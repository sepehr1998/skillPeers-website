<template>
  <div :class="['vue-star-rating', {'vue-star-rating-rtl':rtl}, {'vue-star-rating-inline': inline}]">
    <div
            class="vue-star-rating"
            @mouseleave="resetRating"
    >
      <span
              v-for="n in maxRating"
              :key="n"
              :class="[{'vue-star-rating-pointer': !readOnly }, 'vue-star-rating-star']"
              :style="{'margin-right': margin + 'px'}"
      >
        <star
                :fill="fillLevel[n-1]"
                :size="starSize"
                :points="starPoints"
                :star-id="n"
                :step="step"
                :active-color="activeColor"
                :inactive-color="inactiveColor"
                :border-color="borderColor"
                :active-border-color="userActiveBorderColor"
                :border-width="borderWidth"
                :rounded-corners="roundedCorners"
                :rtl="rtl"
                :glow="glow"
                :glow-color="glowColor"
                :animate="animate"
                @star-selected="setRating($event, true)"
                @star-mouse-move="setRating"
        />
      </span>
      <span
              v-if="showRating"
              :class="['vue-star-rating-rating-text', textClass]"
      > {{ formattedRating }}</span>
    </div>
  </div>
</template>
<script type="text/javascript">
    /* eslint-disable vue/custom-event-name-casing */
    import Star from './components/star.vue'

    export default {

        name: 'StarRating',
        components: {
            Star
        },
        props: {
            increment: {
                type: Number,
                default: 1
            },
            rating: {
                type: Number,
                default: 0
            },
            roundStartRating: {
                type: Boolean,
                default: true
            },
            activeColor: {
                type: String,
                default: '#ffd055'
            },
            inactiveColor: {
                type: String,
                default: '#d8d8d8'
            },
            maxRating: {
                type: Number,
                default: 5
            },
            starPoints: {
                type: Array,
                default() {
                    return []
                }
            },
            starSize: {
                type: Number,
                default: 50
            },
            showRating: {
                type: Boolean,
                default: true
            },
            readOnly: {
                type: Boolean,
                default: false
            },
            textClass: {
                type: String,
                default: ''
            },
            inline: {
                type: Boolean,
                default: false
            },
            borderColor: {
                type: String,
                default: '#999'
            },
            activeBorderColor: {
                type: String,
                default: null
            },
            borderWidth: {
                type: Number,
                default: 0
            },
            roundedCorners: {
                type: Boolean,
                default: false
            },
            padding: {
                type: Number,
                default: 0
            },
            rtl: {
                type: Boolean,
                default: false
            },
            fixedPoints: {
                type: Number,
                default: null
            },
            glow: {
                type: Number,
                default: 0
            },
            glowColor: {
                type: String,
                default: '#fff'
            },
            clearable: {
                type: Boolean,
                default: false
            },
            applyActiveColorOnHover: {
                type: Boolean,
                default: true
            },
            animate: {
                type: Boolean,
                default: false
            }
        },
        emits: ['update:rating', 'hover:rating'],

        data() {
            return {
                step: 0,
                fillLevel: [],
                currentRating: 0,
                selectedRating: 0,
                ratingSelected: false
            }
        },
        computed: {
            formattedRating() {
                return (this.fixedPoints === null) ? this.currentRating : this.currentRating.toFixed(this.fixedPoints)
            },
            shouldRound() {
                return this.ratingSelected || this.roundStartRating
            },
            margin() {
                return this.padding + this.borderWidth
            }
        },
        watch: {
            rating(val) {
                this.currentRating = val
                this.selectedRating = val
                this.createStars(this.shouldRound)
            }
        },
        created() {
            this.step = this.increment * 100
            this.currentRating = this.rating
            this.selectedRating = this.currentRating
            this.userActiveBorderColor = (this.activeBorderColor) ? this.activeBorderColor : this.borderColor
            this.createStars(this.roundStartRating)

        },
        methods: {
            setRating($event, persist) {
                if (!this.readOnly) {
                    const position = (this.rtl) ? (100 - $event.position) / 100 : $event.position / 100
                    this.currentRating = (($event.id + position) - 1).toFixed(2)
                    this.currentRating = (this.currentRating > this.maxRating) ? this.maxRating : this.currentRating
                    if (persist) {
                        this.createStars(true, true)
                        if (this.clearable && this.currentRating === this.selectedRating) {
                            this.selectedRating = 0
                        } else {
                            this.selectedRating = this.currentRating
                        }

                        this.$emit('update:rating', this.selectedRating)
                        this.ratingSelected = true

                    } else {
                        this.createStars(true, this.applyActiveColorOnHover)
                        this.$emit('hover:rating', this.currentRating)
                    }
                }
            },
            resetRating() {
                if (!this.readOnly) {
                    this.currentRating = this.selectedRating
                    this.createStars(this.shouldRound)
                }
            },
            createStars(round = true, applyFill = true) {
                if (round) {
                    this.round()
                }
                for (var i = 0; i < this.maxRating; i++) {
                    let level = 0
                    if (i < this.currentRating) {
                        level = (this.currentRating - i > 1) ? 100 : (this.currentRating - i) * 100
                    }
                    if (applyFill) {
                        this.fillLevel[i] = Math.round(level)
                    }
                }
            },
            round() {
                var inv = 1.0 / this.increment
                this.currentRating = Math.min(this.maxRating, Math.ceil(this.currentRating * inv) / inv)
            }
        }
    }
</script>
<style scoped>
  .vue-star-rating-star {
    display: inline-block;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    -webkit-tap-highlight-color: transparent;
  }

  .vue-star-rating-pointer {
    cursor: pointer;
  }

  .vue-star-rating {
    display: flex;
    align-items: center;
  }

  .vue-star-rating-inline {
    display: inline-flex;
  }


  .vue-star-rating-rating-text {
    margin-left: 7px;
  }

  .vue-star-rating-rtl {
    direction: rtl;
  }

  .vue-star-rating-rtl .vue-star-rating-rating-text {
    margin-right: 10px;
    direction: rtl;
  }
</style>
