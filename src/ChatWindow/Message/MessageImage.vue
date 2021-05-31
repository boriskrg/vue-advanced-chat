<template>
	<div ref="imageRef" class="vac-image-container">
    <div class="vac-message-images-container">
      <div
        v-for="(file, fileIndex) in message.files"
        :key="'message_' + message._id + '_' + fileIndex"
        class="vac-message-image"
        @click.stop="imageClick($event, file, fileIndex)"
        :class="{
				'vac-image-loading':
					isImageLoading(file) && message.senderId === currentUserId
			}"
        :style="{
				'background-image': `url('${imageBackground(file)}')`,
				// 'max-height': `${imageResponsive.maxHeight}px`
			}"
      >
        <loader
          :style="{ top: `${imageResponsive.loaderTop}px` }"
          :show="isImageLoading(file)"
        />
        <transition name="vac-fade-image">
          <div v-if="imageHover && !isImageLoading(file)" class="vac-image-buttons">
            <div
              class="vac-svg-button vac-button-view"
              @click.stop="$event.stopPropagation; $emit('open-file', fileIndex, 'preview')"
            >
              <slot name="eye-icon">
                <svg-icon name="eye" />
              </slot>
            </div>
            <div
              class="vac-svg-button vac-button-download"
              @click.stop="$event.stopPropagation; $emit('open-file', 'download')"
            >
              <slot name="document-icon">
                <svg-icon name="document" />
              </slot>
            </div>
          </div>
        </transition>
      </div>
    </div>
		<format-message
			:content="message.content"
			:users="roomUsers"
			:text-formatting="textFormatting"
			:link-options="linkOptions"
			@open-user-tag="$emit('open-user-tag')"
		>
			<template v-for="(i, name) in $scopedSlots" #[name]="data">
				<slot :name="name" v-bind="data" />
			</template>
		</format-message>
	</div>
</template>

<script>
import Loader from '../../components/Loader'
import SvgIcon from '../../components/SvgIcon'
import FormatMessage from '../../components/FormatMessage'

const { isImageFile } = require('../../utils/media-file')

export default {
	name: 'MessageImage',
	components: { SvgIcon, Loader, FormatMessage },

	props: {
		currentUserId: { type: [String, Number], required: true },
		message: { type: Object, required: true },
		roomUsers: { type: Array, required: true },
		textFormatting: { type: Boolean, required: true },
		linkOptions: { type: Object, required: true },
		imageHover: { type: Boolean, required: true }
	},

	data() {
		return {
			imageLoading: false,
			imageResponsive: ''
		}
	},

	watch: {
		message: {
			immediate: true,
			handler() {
				this.checkImgLoad()
			}
		}
	},

	mounted() {

	},

	methods: {
    // imageResponsive() = {
    //   maxHeight: this.$refs.imageRef.clientWidth - 18,
    //   loaderTop: this.$refs.imageRef.clientWidth / 2
    // },
    isImageLoading(file) {
      return (
        file.url.indexOf('blob:http') !== -1 || this.imageLoading
      )
    },
    imageClick($event, file, fileIndex) {
      $event.stopPropagation()
      this.$emit('open-file', fileIndex, 'image-click')
    },
    imageBackground(file) {
      return this.isImageLoading
        ? file.preview || file.url
        : file.url
    },
		checkImgLoad() {
			if (!isImageFile(this.message.file)) return
			this.imageLoading = true
			const image = new Image()
			image.src = this.message.file.url
			image.addEventListener('load', () => (this.imageLoading = false))
		}
	}
}
</script>

<style lang="scss">
.vac-image-container {
	width: 250px;
	max-width: 100%;

	.vac-image-loading {
		filter: blur(3px);
	}

	.vac-image-buttons {
		position: absolute;
		width: 100%;
		height: 100%;
		border-radius: 4px;
		background: linear-gradient(
			to bottom,
			rgba(0, 0, 0, 0) 55%,
			rgba(0, 0, 0, 0.02) 60%,
			rgba(0, 0, 0, 0.05) 65%,
			rgba(0, 0, 0, 0.1) 70%,
			rgba(0, 0, 0, 0.2) 75%,
			rgba(0, 0, 0, 0.3) 80%,
			rgba(0, 0, 0, 0.5) 85%,
			rgba(0, 0, 0, 0.6) 90%,
			rgba(0, 0, 0, 0.7) 95%,
			rgba(0, 0, 0, 0.8) 100%
		);

		svg {
			height: 26px;
			width: 26px;
		}

		.vac-button-view,
		.vac-button-download {
			position: absolute;
			bottom: 6px;
			left: 7px;
		}

		:first-child {
			left: 40px;
		}

		.vac-button-view {
			max-width: 18px;
			bottom: 8px;
		}
	}
}
</style>
