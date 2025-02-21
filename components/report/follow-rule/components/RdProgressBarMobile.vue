<template>
  <div class="progress-bar" :style="cssProps" :class="{ hide: !shouldShowBar }">
    <div
      class="progress-bar__mobile mobile"
      :class="{ 'hide-title': !isScrollingUp }"
    >
      <div class="animate">
        <RdStalkerAnimation
          :stalkerStatus="stalkerStatus"
          :location="stalkerLocation"
        />
        <RdTrackedAnimation
          v-show="isAnimateFinish < 2"
          :trackedStatus="trackedStatus"
          :location="trackedLocation"
          :isMobile="true"
        />
      </div>
      <div class="mobile__title">
        <div v-for="(row, i) in tagsGroup" :key="i" class="mobile__title_row">
          <div
            v-for="tag in row"
            :key="tag.id"
            class="mobile__title_row_item"
            :class="{ active: parseInt(tag.id) === nowTagId }"
            @click="handleClick(tag.id)"
          >
            {{ tag.title }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
// import rafThrottle from 'raf-throttle'
import RdStalkerAnimation from './RdStalkerAnimation.vue'
import RdTrackedAnimation from './RdTrackedAnimation.vue'
export default {
  components: {
    RdStalkerAnimation,
    RdTrackedAnimation,
  },
  props: {
    tagsArray: {
      type: Array,
      require: true,
      default: () => [],
    },
    isScrollingUp: {
      type: Boolean,
      default: true,
    },
    isScrollEnd: {
      type: Boolean,
      default: false,
    },
    nowTagId: {
      type: Number,
      default: 1,
    },
    shouldShowBar: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      scale: 0.5,
      trackedLocation: 0,
      stalkerLocation: -50,
      trackedStatus: 'stand',
      stalkerStatus: 'stand',
      stalkerMoveId: 0,
      trackedMoveId: 0,
      target: null,
      percent: 0,
      hasFinishedReading: false,
      isAnimateFinish: 0,
      stalkerCanMove: false,
      animateStart: false,
    }
  },
  computed: {
    ...mapGetters('viewport', ['viewportHeight', 'viewportWidth']),
    cssProps() {
      return {
        '--spacing': `${this.spacing}px`,
        '--tracked-location': `${this.trackedLocation}px`,
        '--stalker-location': `${this.stalkerLocation}px`,
        '--scale': this.scale,
      }
    },
    distance() {
      return this.trackedLocation - this.stalkerLocation
    },
    tagsGroup() {
      return [this.tagsArray.slice(0, 2), this.tagsArray.slice(2, 4)]
    },
    minDistance() {
      return parseInt(100 * this.scale)
    },
  },

  watch: {
    distance(d) {
      if (d <= this.minDistance && this.trackedStatus === 'stand') {
        this.trackedStatus = 'afraid'
      }
      if (d === this.minDistance + 1) {
        if (this.trackedStatus !== 'moving') this.trackedStatus = 'stand'
      }
      if (d < this.minDistance) {
        this.handleScroll()
      }
    },
    isScrollEnd() {
      this.endAnimate()
    },
    nowTagId(id) {
      if (id === 1) {
        this.trackedMove(58, 'moving', 50, () => {
          this.trackedStatus = 'stand'
        })
        this.animateStart = true
      }
      if (id > 1) {
        this.stalkerCanMove = true
        this.handleScroll()
      }
    },
  },

  mounted() {
    // this.trackedLocation = this.minDistance
    // this.handleScroll()
    window.addEventListener('scroll', this.throttle(this.handleScroll, 2000))
    window.addEventListener('scroll', this.handleTrackedScroll)
  },

  methods: {
    throttle(callback, limit) {
      let wait = false
      return function () {
        if (!wait) {
          callback()
          wait = true
          setTimeout(function () {
            wait = false
          }, limit)
        }
      }
    },
    handleClick(id) {
      this.$emit('scroll-to-section', id)
    },
    stalkerMove(destination, status, time, cb) {
      if (
        !this.stalkerCanMove ||
        this.isAnimateFinish === 2 ||
        this.stalkerLocation === parseInt(destination)
        // (this.stalkerStatus === 'back' && status === 'back')
        // (status === 'back' && this.stalkerLocation < this.minDistance)
      )
        return
      if (destination < 0) this.stalkerCanMove = false
      this.stalkerMoveId++
      const id = this.stalkerMoveId
      this.stalkerStatus = status
      const interval = setInterval(() => {
        if (id !== this.stalkerMoveId) clearInterval(interval)
        if (this.stalkerLocation === parseInt(destination)) {
          clearInterval(interval)
          cb()
        } else {
          this.stalkerLocation > parseInt(destination)
            ? this.stalkerLocation--
            : this.stalkerLocation++
        }
      }, time)
    },
    trackedMove(destination, status, time, cb) {
      this.trackedMoveId++
      const id = this.trackedMoveId
      this.trackedStatus = status
      const interval = setInterval(() => {
        if (id !== this.trackedMoveId) clearInterval(interval)
        if (this.trackedLocation === parseInt(destination)) {
          clearInterval(interval)
          cb()
        } else {
          this.trackedLocation > parseInt(destination)
            ? this.trackedLocation--
            : this.trackedLocation++
        }
      }, time)
    },
    stalkerForword() {
      if (this.isAnimateFinish === 2) return
      this.stalkerMove(
        this.trackedLocation - this.minDistance,
        'moving',
        20,
        () => {
          this.stalkerStatus = 'stand'
        }
      )
    },
    handleScroll() {
      if (!this.animateStart) return
      this.stalkerMove(0, 'back', 10, () => {
        this.stalkerStatus = 'stand'
        setTimeout(() => this.stalkerForword(), 500)
      })
      if (!this.target) {
        this.target = document.getElementsByTagName('article')[0]
      }
      const { bottom } = this.target.getBoundingClientRect()
      if (bottom - this.viewportHeight < 0) {
        this.percent = 100
        if (this.hasFinishedReading === false) {
          this.hasFinishedReading = true
        }
        return
      }

      const { pageYOffset } = window
      this.percent = Math.round(
        (pageYOffset / (bottom + pageYOffset - this.viewportHeight)) * 100
      )

      const totalWidth = this.viewportWidth - this.minDistance
      const newLocation =
        Math.round(totalWidth * this.percent * 0.01) + this.minDistance
      this.trackedMove(newLocation, 'moving', 10, () => {
        this.trackedStatus = 'stand'
        if (this.stalkerStatus !== 'back') {
          setTimeout(() => this.stalkerForword(), 500)
        }
      })
    },
    handleTrackedScroll() {
      if (!this.target) {
        this.target = document.getElementsByTagName('article')[0]
      }
      const { bottom } = this.target.getBoundingClientRect()
      if (bottom - this.viewportHeight < 0) {
        this.percent = 100
        if (this.hasFinishedReading === false) {
          this.hasFinishedReading = true
        }
        return
      }
      const { pageYOffset } = window
      this.percent = Math.round(
        (pageYOffset / (bottom + pageYOffset - this.viewportHeight)) * 100
      )
      const totalWidth = this.viewportWidth - this.minDistance
      const newLocation =
        Math.round(totalWidth * this.percent * 0.01) + this.minDistance
      this.trackedMove(newLocation, 'moving', 10, () => {
        this.trackedStatus = 'stand'
      })
    },
    endAnimate() {
      this.trackedMove(this.viewportWidth + 50, 'moving', 10, () => {
        this.trackedStatus = 'stand'
        this.isAnimateFinish++
      })
      this.stalkerMove(-50, 'back', 10, () => {
        this.stalkerStatus = 'stand'
        this.isAnimateFinish++
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.progress-bar {
  width: 100%;
  position: sticky;
  top: 0;
  z-index: 20;
  white-space: nowrap;
  &__wrapper {
    width: 712px;
    margin: auto;
  }
}

// .spacer {
//   // height: 163px;
// }

.mobile {
  color: rgba(255, 233, 214, 1);
  font-weight: 600;
  font-size: 14px;
  line-height: 20px;

  &__title {
    background: #000000;
    padding: 11px 28px;
    .active {
      color: #28ddb1;
    }
    &_row {
      display: flex;
      justify-content: space-around;
      &:not(:first-child) {
        margin-top: 11px;
      }
      * {
        display: flex;
        &:first-child {
          width: 50%;
          justify-content: center;
        }
        &:not(:first-child) {
          flex: 1;
          margin-left: 10%;
        }
      }
    }
  }

  .animate {
    display: flex;
    position: relative;
    // height: 90px;
    overflow: hidden;
    background: #feeade;
    & > div {
      width: calc(150px * 0.5 * var(--scale));
      height: calc(230px * 0.5 * var(--scale));
      transform: translate(-50%, 0);
    }

    &::before {
      content: '';
      background: #28ddb1;
      height: 100%;
      width: var(--tracked-location);
      position: absolute;
    }
  }
}

.hide-title {
  .mobile__title {
    opacity: 0;
  }
  a {
    cursor: default;
    z-index: -100;
  }
}

.hide {
  opacity: 0;
  a {
    cursor: default;
    z-index: -100;
  }
}
</style>
