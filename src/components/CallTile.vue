<template>
  <main>
    <!-- loading is true when the call is in the "joining-meeting" meeting state -->
    <template v-if="loading">
      <div class="loading-spinner"><loading-tile /></div>
    </template>

    <template v-else>
      <div class="wrapper">
        <template v-if="error">
          <p class="error-text">{{ error }}</p>
          <!-- refreshing will leave the call and reset the app state -->
          <button class="error-button" @click="leaveAndCleanUp">Refresh</button>
        </template>

        <template v-if="showPermissionsError">
          <permissions-error-msg :reset="leaveAndCleanUp" />
        </template>

        <template v-else>
          <div
            :class="screen ? 'tile-container' : 'tile-container full-height'"
          >
            <template v-if="screen">
              <screenshare-tile :participant="screen" />
            </template>

            <div v-if="participants" class="participants-container">
              <div id="video-call" ref="videoCall"></div>

              <template v-if="count === 1">
                <waiting-card :url="roomUrl" />
              </template>
            </div>
          </div>
        </template>

        <chat-tile :send-message="sendMessage" :messages="messages" />
      </div>
    </template>
  </main>
</template>

<script lang="ts">
import { defineComponent } from "vue";

import daily, {
  type DailyCall,
  type DailyEventObjectAppMessage,
  type DailyEventObjectFatalError,
  type DailyParticipant,
} from "@daily-co/daily-js";

import WaitingCard from "./WaitingCard.vue";
import ChatTile from "./ChatTile.vue";
import ScreenshareTile from "./ScreenshareTile.vue";
import LoadingTile from "./LoadingTile.vue";
import PermissionsErrorMsg from "./PermissionsErrorMsg.vue";

interface Participant {
  session_id: string;
  user_id: string;
  user_name: string;
  local: boolean;
  video: boolean;
  audio: boolean;
  screen: boolean;
}

interface Message {
  name: string;
  message: string;
}

interface CallTileData {
  callObject: null | DailyCall;
  loading: boolean;
  error: string;
  showPermissionsError: boolean;
  participants: Participant[];
  screen: unknown; // video track
  messages: Message[];
  count: number;
}

type Tracks = {
  videoTrack: MediaStreamTrack | null;
  audioTrack: MediaStreamTrack | null;
};

export default defineComponent({
  name: "CallTile",
  components: {
    WaitingCard,
    ChatTile,
    ScreenshareTile,
    LoadingTile,
    PermissionsErrorMsg,
  },
  props: {
    leaveCall: {
      type: Function,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    roomUrl: {
      type: String,
      required: true,
    },
  },
  data(): CallTileData {
    return {
      callObject: null,
      participants: [],
      count: 0,
      messages: [],
      error: '',
      loading: false,
      showPermissionsError: false,
      screen: null,
    };
  },
  mounted() {
    const option = {
      url: this.roomUrl,
    };

    // Create instance of Daily call object
    const co = daily.createCallObject(option);
    // Assign in data obj for future reference
    this.callObject = co;

    // Join the call with the name set in the Home.vue form
    co.join({ userName: this.name });

    // Add call and participant event handler
    // Visit https://docs.daily.co/reference/daily-js/events for more event info
    co
      .on("error", this.handleError)
      // camera-error = device permissions issue
      .on("camera-error", this.handleDeviceError)
      // app-message handles receiving remote chat messages
      .on("app-message", this.updateMessages)
      .on("track-started", (p) => {
        if (!p?.participant) return;
        const tracks = this.getParticipantTracks(p.participant);
        try {
          this.updateMedia(p.participant.session_id, tracks);
        } catch (e) {
          console.warn(e);
        }
      })
      .on("track-stopped", (p) => {
        if (!p?.participant) return;
        const tracks = this.getParticipantTracks(p.participant);
        try {
          this.updateMedia(p.participant.session_id, tracks);
        } catch (e) {
          console.warn(e);
        }
      });
  },
  unmounted() {
    if (!this.callObject) return;
    // Clean-up event handlers
    this.callObject
      .off("error", this.handleError)
      .off("camera-error", this.handleDeviceError)
      .off("app-message", this.updateMessages)
      .off("track-started", (p) => {
        if (!p?.participant) return;
        const tracks = this.getParticipantTracks(p.participant);
        try {
          this.updateMedia(p.participant.session_id, tracks);
        } catch (e) {
          console.warn(e);
        }
      })
      .off("track-stopped", (p) => {
        if (!p?.participant) return;
        const tracks = this.getParticipantTracks(p.participant);
        try {
          this.updateMedia(p.participant.session_id, tracks);
        } catch (e) {
          console.warn(e);
        }
      });
  },
  methods: {
    updateMedia(participantID: string, newTracks: Tracks) {
      // Get the video tag.
      let videoTile = document.getElementById(
        participantID
      ) as HTMLVideoElement | null;
      if (!videoTile) {
        const videoCall = this.$refs.videoCall as HTMLVideoElement;
        
        const newVideoTile = document.createElement("video");
        newVideoTile.id = participantID;
        videoCall.appendChild(newVideoTile);
        videoTile = newVideoTile;
      }

      const video = videoTile;

      // Get existing MediaStream from the video tag source object.
      const existingStream = video.srcObject as MediaStream;

      const newVideo = newTracks.videoTrack;
      const newAudio = newTracks.audioTrack;

      // If there is no existing stream or it contains no tracks,
      // Just create a new media stream using our new tracks.
      // This will happen if this is the first time we're
      // setting the tracks.
      if (!existingStream || existingStream.getTracks().length === 0) {
        const tracks: MediaStreamTrack[] = [];
        if (newVideo) tracks.push(newVideo);
        if (newAudio) tracks.push(newAudio);
        const newStream = new MediaStream(tracks);
        video.srcObject = newStream;
        video.playsInline = true;
        video.autoplay = true;
        video.muted = true;
        this.playMedia(video);
        return;
      }

      // This boolean will define whether we play the video element again
      // This should be `true` if any of the tracks have changed.
      let needsPlay = false;
      needsPlay = this.refreshAudioTrack(existingStream, newAudio);

      // We have an extra if check here compared to the audio track
      // handling above, because the video track also dictates
      // whether we should hide the video DOM element.
      if (newVideo) {
        if (this.refreshVideoTrack(existingStream, newVideo) && !needsPlay) {
          needsPlay = true;
        }

        video.classList.remove("hidden");
      } else {
        // If there's no video to be played, hide the element.
        video.classList.add("hidden");
      }
      if (needsPlay) {
        this.playMedia(video);
      }
    },
    refreshAudioTrack(
      existingStream: MediaStream,
      newAudioTrack: MediaStreamTrack | null
    ): boolean {
      // If there is no new track, just early out
      // and keep the old track on the stream as-is.
      if (!newAudioTrack) return false;
      const existingTracks = existingStream.getAudioTracks();
      return this.refreshTrack(existingStream, existingTracks, newAudioTrack);
    },
    refreshVideoTrack(
      existingStream: MediaStream,
      newVideoTrack: MediaStreamTrack | null
    ): boolean {
      // If there is no new track, just early out
      // and keep the old track on the stream as-is.
      if (!newVideoTrack) return false;
      const existingTracks = existingStream.getVideoTracks();
      return this.refreshTrack(existingStream, existingTracks, newVideoTrack);
    },
    refreshTrack(
      existingStream: MediaStream,
      oldTracks: MediaStreamTrack[],
      newTrack: MediaStreamTrack
    ): boolean {
      const trackCount = oldTracks.length;
      // If there is no matching old track,
      // just add the new track.
      if (trackCount === 0) {
        existingStream.addTrack(newTrack);
        return true;
      }
      if (trackCount > 1) {
        console.warn(
          `expected up to 1 media track, but got ${trackCount}. Only using the first one.`
        );
      }
      const oldTrack = oldTracks[0];
      // If the IDs of the old and new track don't match,
      // replace the old track with the new one.
      if (oldTrack.id !== newTrack.id) {
        existingStream.removeTrack(oldTrack);
        existingStream.addTrack(newTrack);
        return true;
      }
      return false;
    },
    playMedia(video: HTMLVideoElement) {
      const isPlaying =
        !video.paused &&
        !video.ended &&
        video.currentTime > 0 &&
        video.readyState > video.HAVE_CURRENT_DATA;

      if (isPlaying) return;

      video.play().catch((e) => {
        if (e instanceof Error && e.name === "NotAllowedError") {
          throw new Error("Autoplay error");
        }

        console.warn("Failed to play media.", e);
      });
    },
    getParticipantTracks(p: DailyParticipant) {
      const mediaTracks: Tracks = {
        videoTrack: null,
        audioTrack: null,
      };

      const tracks = p?.tracks;
      if (!tracks) return mediaTracks;

      const vt = tracks.video;
      const vs = vt?.state;
      if (vt.persistentTrack && !(vs === "off" || vs === "blocked")) {
        mediaTracks.videoTrack = vt.persistentTrack;
      }

      // Only get audio track if this is a remote participant
      if (!p.local) {
        const at = tracks.audio;
        const as = at?.state;
        if (at.persistentTrack && !(as === "off" || as === "blocked")) {
          mediaTracks.audioTrack = at.persistentTrack;
        }
      }
      return mediaTracks;
    },
    /**
     * This is called any time a participant update registers.
     * In large calls, this should be optimized to avoid re-renders.
     * For example, track-started and track-stopped can be used
     * to register only video/audio/screen track changes.
     */
    updateParticpants(e) {
      console.log("[EVENT] ", e);
      if (!this.callObject) return;

      const p = this.callObject.participants();
      this.count = Object.values(p).length;
      this.participants = Object.values(p);

      const screen = this.participants.filter((pa) => pa.screenVideoTrack);
      if (screen?.length && !this.screen) {
        console.log("[SCREEN]", screen);
        this.screen = screen[0];
      } else if (!screen?.length && this.screen) {
        this.screen = null;
      }
      this.loading = false;
    },
    // Add chat message to local message array
    updateMessages(e: DailyEventObjectAppMessage<any> | undefined) {
      if (!e) return;
      console.log("[MESSAGE] ", e.data);
      this.messages.push(e?.data);
    },
    // Show local error in UI when daily-js reports an error
    handleError(e: DailyEventObjectFatalError | undefined) {
      if (!e) return;
      console.log("[ERROR] ", e);
      this.error = e.errorMsg;
      this.loading = false;
    },
    // Temporary show loading view while joining the call
    handleJoiningMeeting() {
      this.loading = true;
    },
    // Toggle local microphone in use (on/off)
    handleAudioClick() {
      const audioOn = this.callObject?.localAudio();
      this.callObject?.setLocalAudio(!audioOn);
    },
    // Toggle local camera in use (on/off)
    handleVideoClick() {
      const videoOn = this.callObject?.localVideo();
      this.callObject?.setLocalVideo(!videoOn);
    },
    // Show permissions error in UI to alert local participant
    handleDeviceError() {
      this.showPermissionsError = true;
    },
    // Toggle screen share
    handleScreenshareClick() {
      if (this.screen?.local) {
        this.callObject?.stopScreenShare();
        this.screen = null;
      } else {
        this.callObject?.startScreenShare();
      }
    },
    /**
     * Send broadcast message to all remote call participants.
     * The local participant updates their own message history
     * because they do no receive an app-message Daily event for their
     * own messages.
     */
    sendMessage(text: string) {
      if (!this.callObject) return;
      // Attach the local participant's username to the message to be displayed in ChatTile.vue
      const local = this.callObject.participants().local;
      const message = { message: text, name: local?.user_name || "Guest" };
      this.messages.push(message);
      this.callObject.sendAppMessage(message, "*");
    },
    // leave call, destroy call object, and reset local state values
    leaveAndCleanUp() {
      if (!this.callObject) return;
      if (this.screen?.local) {
        this.callObject.stopScreenShare();
      }
      this.callObject.leave().then(() => {
        this.callObject?.destroy();

        this.participantWithScreenshare = null;
        this.screen = null;
        this.leaveCall();
      });
    },
  },
});
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Ropa+Sans&display=swap");
main {
  font-family: "Ropa Sans", sans-serif;
  background-color: #121a24;
  height: 100%;
  position: relative;
}
.wrapper {
  max-width: 1200px;
  margin: auto;
  padding: 0 16px;
  height: 100%;
}
.loading-spinner {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}
.tile-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.participants-container {
  display: flex;
  margin: 0 -20px;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  width: 100%;
  background-color: #121a24;
  height: inherit;
}
p {
  color: white;
}
.error-text {
  color: #e71115;
}
.full-height {
  height: 100%;
}
.error-button {
  color: #fff;
  background-color: #121a24;
  border: none;
  font-size: 12px;
  border: 1px solid #2b3f56;
  border-radius: 8px;
  padding: 8px 12px;
  cursor: pointer;
}

#video-call {
  min-width: 50%;
}
</style>
