<template>
    <section class="app-popup">
        <transition name="scale">
            <div
                    class="app-popup__wrapper padding--m"
                    :class="{ 'app-popup__wrapper--flex-top': isPopupOverWindow }"
                    v-if="isPopupOpen"
                    @click.self="onClosePopup(currentPopup)"
                    ref="popupWrapper"
            >
                <div class="app-popup__popup" ref="popup">
                    <keep-alive>
                        <component :is="currentPopupComponent" @close-popup="closePopup" @toggle-popup="togglePopup" />
                    </keep-alive>
                </div>
            </div>
        </transition>
    </section>
</template>
<script>
    import PopupSuccess from "./PopupSuccess";
    import PopupError from "./PopupError";

    export default {
        data() {
            return {
                currentPopup: '',
                popups: ['popup-success', 'popup-error'],
                isPopupOpen: false,
                isPopupOverWindow: false,
                timer: null,
            }
        },
        components: {
            PopupSuccess,
            PopupError,
        },
        computed: {
            currentPopupComponent() {
                return this.currentPopup;
            },
        },
        methods: {
            openPopup(name) {
                const popup = this.popups.find(item => item === name) || '';
                this.isPopupOpen = !!popup;
                this.currentPopup = popup;
                this.setPosition();
            },
            closePopup() {
                this.isPopupOpen = false;

                this.currentPopup = '';
            },
            onClosePopup(name) {
                this.$bus.$emit('app-close-popup', name);
            },
            togglePopup(name) {
                this.closePopup();
                this.openPopup(name);
            },
            setPosition() {
                if (this.timer) {
                    clearTimeout(this.timer);
                    this.timer = null;
                }

                const ctx = this;

                const callback = () => {
                    const popup = ctx.$refs.popup;
                    if (popup) {
                        const popupWrapper = this.$refs.popupWrapper;
                        if (popup.clientHeight > popupWrapper.clientHeight) ctx.isPopupOverWindow = true;
                        else ctx.isPopupOverWindow = false;
                        clearTimeout(ctx.timer);
                        ctx.timer = null;
                    }
                };
                this.timer = setTimeout(callback, 250)
            },
        },
        beforeMount() {
            window.addEventListener('resize', this.setPosition);
        },
        created() {
            this.$bus.$on('app-open-popup', this.openPopup);
            this.$bus.$on('app-close-popup', this.closePopup);
        },
        beforeDestroy() {
            this.$bus.$off('app-open-popup', this.openPopup);
            this.$bus.$off('app-close-popup', this.closePopup);
            window.removeEventListener('resize', this.setPosition);
        },
    }
</script>
<style lang="scss">
    .app-popup {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        padding-right: 15px;
        z-index: 200;
        pointer-events: none;
        overflow: hidden;
        min-width: 320px;

        @include w-to($screen-lg) {
            padding-right: 0;
        }

        &__wrapper {
            position: fixed;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0 -15px 0 0;
            left: 0;
            top: 0;
            bottom: 0;
            right: 0;
            background-color: rgba(49, 49, 49, 0.75);
            pointer-events: auto;
            overflow: auto;

            &--flex-top {
                align-items: flex-start;
            }

            @include w-to($screen-lg) {
                margin: 0;
            }
        }

        &__popup {
            flex-shrink: 0;
            max-width: 100%;
            position: relative;
            transition-property: transform, opacity;
            transition-duration: 500ms;
            //box-shadow: var(--global-box-shadow-popup);
        }

        &__close {
            position: absolute;
            top: 20px;
            right: 20px;
            width: 50px;
            height: 50px;
            background-color: transparent;
            box-shadow: none;
            border: none;
            transition: $transition-normal;
            opacity: 0.5;
            cursor: pointer;

            &::before,
            &::after {
                content: '';
                display: block;
                position: absolute;
                top: 50%;
                left: 50%;
                width: 15px;
                height: 2px;
                //background-color: var(--global-color);
                transform: translate(-50%, -50%);
                pointer-events: none;
            }
            &::before {
                transform: translate(-50%, -50%) rotate(-45deg);
            }
            &::after {
                transform: translate(-50%, -50%) rotate(45deg);
            }
            &:hover,
            &:focus {
                transition: $transition-normal;
                opacity: 0.8;
            }
        }

        .scale-enter-active,
        .scale-leave-active {
            opacity: 1;
            transition-duration: 500ms;
            .app-popup__popup {
                transition-duration: 300ms;
                transform: scale(1);
            }
        }

        .scale-enter,
        .scale-leave-to {
            opacity: 0;
            transition-duration: 500ms;
            .app-popup__popup {
                transform: scale(0.2);
            }
        }
    }
</style>
