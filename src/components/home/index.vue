<template>
  <WebGl />
  <div id="home">
    <div class="row">
      <div class="d-flex">
        <div class="icons">
          <Logo />
          <div class="media-icons">
            <div class="social-icons"><Twitter /></div>
            <div class="social-icons"><Instagram /></div>
            <div class="social-icons"><Facebook /></div>
          </div>
          <div></div>
        </div>
        <div class="body">
          <div class="demo-reel">
            <div class="d-flex">
              <LeftArrow />
              <div class="demo-reel-text">Our Show Reel</div>
            </div>
          </div>
          <div class="home-text">
            <div></div>
            <div class="row mx-0">
              <div class="col-md-1"></div>
              <div class="col-md-6">
                <div class="welcom-text">
                  We are guez agency and we are great <span>strategists</span>
                </div>
              </div>
            </div>
            <div class="scroll-info">
              <div class="row mx-0">
                <div class="col-md-1"></div>
                <div class="col-md-1">
                  <div class="text">Scroll Down</div>
                  <DownArrow />
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="menu">
          <div class="menu-des">
            <div class="line-1"></div>
            <div class="line-2"></div>
            <div class="line-3"></div>
          </div>
          <div class="text">Menu</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Logo from "./src/Logo";
import Twitter from "./src/Twitter.vue";
import Instagram from "./src/Instagram.vue";
import Facebook from "./src/Facebook.vue";
import LeftArrow from "./src/LeftArrow.vue";
import DownArrow from "./src/ArrowDown.vue";

//webgl
import WebGl from "../webgl/index.vue";

export default {
  components: {
    Logo: Logo,
    Twitter: Twitter,
    Instagram: Instagram,
    Facebook: Facebook,
    LeftArrow,
    DownArrow,
    WebGl,
  },
  name: "Home",
  props: {
    title: String,
    mission: String,
    missionDetails: String,
  },
};
</script>
<style lang="scss">
@import "./index.scss";
</style>
