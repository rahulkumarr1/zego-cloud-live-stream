<template>
  <div id="app" ref="root"></div>
</template>

<script>
import { ZegoUIKitPrebuilt } from '@zegocloud/zego-uikit-prebuilt';

// get token
function generateToken(tokenServerUrl, userID) {
  // Obtain the token interface provided by the App Server
  return fetch(
    `${tokenServerUrl}/zego_access_token?userID=${userID}&expired_ts=7200`,
    {
      method: 'GET',
    }
  ).then((res) => res.json());
}

function randomID(len) {
  let result = '';
  if (result) return result;
  var chars = '12345qwertyuiopasdfgh67890jklmnbvcxzMNBVCZXASDQWERTYHGFUIOLKJP';
  var maxPos = chars.length;
  var i;
  len = len || 5;
  for (i = 0; i < len; i++) {
    result += chars.charAt(Math.floor(Math.random() * maxPos));
  }
  return result;
}

export function getUrlParams(url) {
  let urlStr = url.split('?')[1];
  return new URLSearchParams(urlStr);
}

export default {
  name: 'App',
  components: {},
  mounted() {
    const roomID = getUrlParams().get('roomID') || randomID(5);
    const userID = randomID(5);
    const userName = randomID(5);
    let role_str = getUrlParams(window.location.href).get('role') || 'Host';
    const role =
      role_str === 'Host'
        ? ZegoUIKitPrebuilt.Host
        : role_str === 'Cohost'
          ? ZegoUIKitPrebuilt.Cohost
          : ZegoUIKitPrebuilt.Audience;

    let sharedLinks = [];
    if (role === ZegoUIKitPrebuilt.Host || role === ZegoUIKitPrebuilt.Cohost) {
      sharedLinks.push({
        name: 'Join as co-host',
        url:
          window.location.origin +
          window.location.pathname +
          '?roomID=' +
          roomID +
          '&role=Cohost',
      });
    }
    sharedLinks.push({
      name: 'Join as audience',
      url:
        window.location.origin +
        window.location.pathname +
        '?roomID=' +
        roomID +
        '&role=Audience',
    });

    // generate token
    generateToken('https://dhwaniastro.democlicks.com/', userID).then((res) => {

      // const appID = 669141563;
      // const serverSecret = "ba36362349d9987625c62cb8030c7683";
      // const token = ZegoUIKitPrebuilt.generateKitTokenForTest(appID, serverSecret, roomID, userID, userName);

      const token = ZegoUIKitPrebuilt.generateKitTokenForProduction(
        669141563,
        res._token,
        roomID,
        userID,
        userName
      );

      // create instance object from token
      const zp = ZegoUIKitPrebuilt.create(token);
      // start the call
      zp.joinRoom({
        container: this.$refs.root,
        scenario: {
          mode: ZegoUIKitPrebuilt.LiveStreaming,
          config: {
            role,
          },
        },
        sharedLinks,
      });
    });
  },
};
</script>

<style>
#app {
  height: 100vh;
  width: 100vw;
}
</style>
