<script>
// Dominik Czechowski
// Calculations and some properties based on https://github.com/denilsonsa/html5-knob/blob/master/xknob.js

export default {
    name: "InfiniteKnob",
    emits: ['new-value'],
    props: {
        min: {
            type: Number,
            default: -0.375,
        },
        max: {
            type: Number,
            default: 0.375,
        },
        divisions: {
            type: Number,
            default: null,
        },
        updateSpeed: {
            type: Number,
            default: 100,
        },
    },
    data() {
        return {
            elem: null,
            timer: null,
            captureToggle: false,
            centerX: 0,
            centerY: 0,
            _value: 0,
            drag_previous_angle: 0,
            drag_previous_value: 0,
            drag_initial_value: 0,
        };
    },
    computed: {
        value: {
            get() {
                return this._value
            },
            set(newValue) {
                this._value = newValue
                if (!Number.isFinite(this._value)) {
                    this._value = 0;
                }
                if (Number.isFinite(this.divisions) && this.divisions >= 2) {
                    this._value = Math.round(this._value * this.divisions) / this.divisions;
                }
                if (Number.isFinite(this.max) && this._value > this.max) {
                    this._value = this.max;
                }
                if (Number.isFinite(this.min) && this._value < this.min) {
                    this._value = this.min;
                }

                if (this.captureToggle) {
                    this.drag_previous_value = this._value;
                }

                this.updateGfxRotation();
                this.timer = setInterval(() => {
                    if (this.captureToggle && this.value != 0) {
                        this.emitNewValue(this.value)
                    }
                }, this.updateSpeed);
            }
        }
    },
    mounted() {
        window.addEventListener('mouseup', this.onMouseUp, false);
    },
    methods: {
        updateGfxRotation() {
            if (this.elem) {
                this.elem.style.transform = 'rotate(' + (this.value * 360) + 'deg)';
            }
        },
        onMouseMove: function (event) {
            if (this.captureToggle) {
                var new_angle = this.getMouseAngle(event);
                var old_angle = this.drag_previous_angle;
                this.drag_previous_angle = new_angle;

                var delta_angle = new_angle - old_angle;
                if (delta_angle < 0) {
                    // Because this is a circle
                    delta_angle += Math.PI * 2;
                }
                if (delta_angle > Math.PI) {
                    // Converting from 0..360 to -180..180.
                    delta_angle -= Math.PI * 2;
                }
                var delta_value = delta_angle / Math.PI / 2;
                var new_proposed_value = this.drag_previous_value + delta_value;
                var old_actual_value = this.value;

                this.value = new_proposed_value;

                this.drag_previous_value = new_proposed_value;

                if (old_actual_value !== new_proposed_value) {
                    this.emitNewValue(new_proposed_value)
                }

            } else {
                clearInterval(this.timer)
                this.value = 0
            }
        },
        emitNewValue(newValue) {
            this.$emit('new-value', newValue)
        },
        onMouseDown: function (event) {
            this.captureToggle = true
            this.elem = event.target;
            const rect = this.elem.getBoundingClientRect();

            this.centerX = rect.left + rect.width / 2;
            this.centerY = rect.top + rect.height / 2;
            this.drag_previous_angle = this.getMouseAngle(event);
            this.drag_previous_value = this.value;
            this.drag_initial_value = this.value;
        },
        onMouseUp: function (event) {
            this.captureToggle = false
            this.value = 0
        },
        getMouseAngle: function (event) {
            let cursor = { x: event.clientX, y: event.clientY };
            if (event.targetTouches && event.targetTouches[0]) {
                cursor.x = event.targetTouches[0].clientX
                cursor.y = event.targetTouches[0].clientY
            }

            const delta = { x: cursor.x - this.centerX, y: cursor.y - this.centerY }
            var rad = Math.atan2(delta.y, delta.x);
            rad += Math.PI / 2;
            return rad;
        },
    }
}
</script>

<template >
    <div class="infinite-knob-background">
        <div class="infinite-knob" @mousedown.self="onMouseDown" @mousemove.self="onMouseMove">
            <div class="infinite-knob-marker"></div>
        </div>
    </div>
</template>

<style lang="scss">
* {
    box-sizing: border-box;
}

.infinite-knob-background {
    width: 120px;
    height: 120px;
    // background: conic-gradient(from 180deg at 50% 33%, #0000ff, rgba(255, 255, 255, 1), #ff0000, from 135deg #fff);
    background: rgb(255, 255, 255);
    border-radius: 50%;
    background: conic-gradient(from 218deg at 50% 65%, rgba(255, 255, 255, 1), rgba(0, 0, 255, 1), rgba(255, 255, 255, 0), rgba(255, 0, 0, 1), rgba(255, 255, 255, 1), rgba(255, 255, 255, 1));
    outline: 4px solid #fff;
    outline-offset: -10px;
}

.infinite-knob {
    background-color: #ddd;
    transform: rotate(0);
    transform-origin: center;
    user-select: none;
    touch-action: none;
    // border: 1px solid rgba(0,0,0, 0.4);
    border-radius: 50%;
    display: block;
    width: 100px;
    height: 100px;
    transition: .3s ease-in-out;
    position: relative;
    left: 10px;
    top: 10px;
    box-shadow: 0px 0px 3px 0px rgba(0, 0, 0, 0.9);

    .infinite-knob-marker {
        position: absolute;
        // background: rgba(0, 0, 0, 0.4);
        box-shadow: inset 0px 0px 3px 0px rgba(0, 0, 0, 0.8);
        border-radius: 2px;
        height: 15px;
        top: 3px;
        left: 50%;
        width: 3px;
    }
}
</style>