<template>
	<div
		class="resizable resizable-container"
		:style="{
			top:withUnit(top),
			left:withUnit(left),
			width:withUnit(width),
			height:withUnit(height),
			'--resizable-handler-size': withUnit(handlerSize)
    }"
	>
		<div class="resizable__handler resizable__handler-corner resizable__handler-corner__top-left"></div>
		<div class="resizable__handler resizable__handler-corner resizable__handler-corner__top-right"></div>
		<div class="resizable__handler resizable__handler-corner resizable__handler-corner__bottom-right"></div>
		<div class="resizable__handler resizable__handler-corner resizable__handler-corner__bottom-left"></div>

		<div class="resizable__handler resizable__handler-edge resizable__handler-edge__top"></div>
		<div class="resizable__handler resizable__handler-edge resizable__handler-edge__right"></div>
		<div class="resizable__handler resizable__handler-edge resizable__handler-edge__bottom"></div>
		<div class="resizable__handler resizable__handler-edge resizable__handler-edge__left"></div>

		<div class="resizable__content">
			<slot></slot>
		</div>
	</div>
</template>

<script>

const edgePositions = ['t', 'r', 'b', 'l']
const directions = ['tl', 'tr', 'br', 'bl']
const units = ['px', 'em', 'rem', '%', 'vh', 'vw']

export const miniId = (len = 6) => {
	return Math.random().toString(36).slice(len);
}

export default {
	name: 'Resizable',
	props: {
		top: {
			default: 100,
			type: [Number, String]
		},
		left: {
			default: 100,
			type: [Number, String]
		},
		width: {
			default: 100,
			type: [Number, String]
		},
		height: {
			default: 100,
			type: [Number, String]
		},
		handlerSize: {
			default: 2,
			type: [Number, String]
		}
	},
	data: () => ({
		id: miniId(),

		mWidth: 0,
		mHeight: 0,

		posX: 0,
		posY: 0,
		mouseX: 0,
		mouseY: 0,
		minSize: 100,
		element: null,

		activeHandler: {
			el: null, type: null, dir: null
		}
	}),
	created() {
		this.mWidth = this.withoutUnit(this.width)
		this.mHeight = this.withoutUnit(this.height)
	},
	mounted() {
		window.addEventListener('mouseup', this.onMouseUp)

		const corners = this.getElements('.resizable__handler-corner')
		corners.forEach((corner, cornerIndex) => {
			corner.addEventListener('mouseup', this.onMouseUp)
			corner.addEventListener('mousedown', async e => {
				e.preventDefault();

				this.activeHandler = {
					el: corner, type: 'corner',
					dir: directions[cornerIndex]
				}

				const rect = await this.$el.getBoundingClientRect()

				this.posX = rect.left; this.posY = rect.top;
				this.mouseX = e.pageX; this.mouseY = e.pageY;

				window.addEventListener("mousemove", this.onMouseMove);
			})
		})

		const edges = this.getElements('.resizable__handler-edge')
		edges.forEach((edge, edgeIndex) => {
			edge.addEventListener('mouseup', this.onMouseUp)
			edge.addEventListener('mousedown', async e => {
				e.preventDefault();

				this.activeHandler = {
					el: edge, type: 'edge',
					dir: edgePositions[edgeIndex]
				}

				const rect = await this.$el.getBoundingClientRect()

				this.posX = rect.left; this.posY = rect.top;
				this.mouseX = e.pageX; this.mouseY = e.pageY;

				window.addEventListener("mousemove", this.onMouseMove);
			})
		})

	},
	beforeUnmount() {
		window.removeEventListener('mouseup', this.onMouseUp)
	},
	watch: {
		width(v) { this.mWidth = this.withoutUnit(v) },
		height(v) { this.mHeight = this.withoutUnit(v) }
	},
	methods: {
		getElements(selector) {
			return this.$el.querySelectorAll(selector)
		},
		withUnit(size) {
			if (typeof size === 'number') return `${size}px`
			const hasUnit = units.some(unit => size.endsWith(unit))
			return hasUnit ? size : `${size}px`
		},
		withoutUnit(size) {
			if (typeof size === 'number') return size
			const unit = units.find(unit => size.endsWith(unit))
			return unit ? +size.replace(unit, '') : +size
		},
		setStyle(style) {
			Object.entries(style).forEach(([property, value]) => {
				this.$el.style[property] = this.withUnit(value)
			})
		},
		onMouseMove(e) {

			const { dir, type } = this.activeHandler

			let height = 0, width = 0, diffX = e.pageX - this.mouseX, diffY = e.pageY - this.mouseY

			if (type === 'corner') {
				width = ['br', 'tr'].includes(dir) ? this.mWidth + diffX : this.mWidth - diffX
				height = ['br', 'bl'].includes(dir) ? this.mHeight + diffY : this.mHeight - diffY
			}
			else if (type === 'edge') {
				width = dir === 'r' ? this.mWidth + diffX : dir === 'l' ? this.mWidth - diffX : width
				height = dir === 'b' ? this.mHeight + diffY : dir === 't' ? this.mHeight - diffY : height
			}

			let canIncrHeight = height > this.minSize
			let canIncWidth = width > this.minSize

			if (dir === 't' && canIncrHeight) {
				this.setStyle({ height, top: this.posY + diffY })
			}
			else if (dir === 'r' && canIncWidth) {
				this.setStyle({ width })
			}
			else if (dir === 'b' && canIncrHeight) {
				this.setStyle({ height })
			}
			else if (dir === 'l' && canIncWidth) {
				this.setStyle({
					width, left: this.posX + diffX
				})
			}
			else if (dir === 'br') {
				if (canIncWidth) this.setStyle({ width })
				if (canIncrHeight) this.setStyle({ height })
			}
			else if (dir === "bl") {
				if (canIncrHeight) this.setStyle({ height })
				if (canIncWidth) this.setStyle({
					width, left: this.posX + diffX
				})

			}
			else if (dir === "tr") {
				if (canIncWidth) this.setStyle({ width })
				if (canIncrHeight) this.setStyle({
					height, top: this.posY + diffY
				})
			}
			else {
				if (canIncWidth) this.setStyle({
					width, left: this.posX + diffX
				})
				if (canIncrHeight) this.setStyle({
					height, top: this.posY + diffY
				})
			}
		},
		onMouseUp() {
			this.activeHandler = { el: null, type: null, dir: null }
			this.mWidth = this.withoutUnit(this.$el.style.width)
			this.mHeight = this.withoutUnit(this.$el.style.height)
			window.removeEventListener('mousemove', this.onMouseMove)
		}
	}
}
</script>

<style scoped lang='scss'>
	.resizable {
		position: absolute;
		background-color: black;
		&__content {
			position: absolute;
			top: var(--resizable-handler-size);
			right: var(--resizable-handler-size);
			bottom: var(--resizable-handler-size);
			left: var(--resizable-handler-size);
		}
		&__handler {
			position: absolute;
		}
		&__handler-corner {
			width: 0px;
			height: 0px;
			z-index: 10;
			border-style: solid;
			border-color: transparent;
			border-width: calc(var(--resizable-handler-size) * 1.5);
			border-left-width: 0px;
			&__top-left,
			&__top-right {
				top: calc(var(--resizable-handler-size) * -1);
			}
			&__bottom-left,
			&__bottom-right {
				bottom: calc(var(--resizable-handler-size) * -1);
			}
			&__top-left,
			&__bottom-left {
				left: calc(calc(var(--resizable-handler-size) / 4) * -1);
			}
			&__top-right,
			&__bottom-right {
				right: calc(calc(var(--resizable-handler-size) / 4) * -1);
			}
			&__top-left,
			&__bottom-right {
				cursor: nw-resize;
			}
			&__top-right,
			&__bottom-left {
				cursor: ne-resize;
			}
			&__top-left {
				transform: rotate(45deg);
			}
			&__top-right {
				transform: rotate(135deg);
			}
			&__bottom-left {
				transform: rotate(-45deg);
			}
			&__bottom-right {
				transform: rotate(-135deg);
			}
		}
		&__handler-edge {
			&__top {
				top: 0;
			}
			&__right {
				right: 0;
			}
			&__bottom {
				bottom: 0;
			}
			&__left {
				left: 0;
			}
			&__top,
			&__bottom {
				left: 0;
				right: 0;
				cursor: n-resize;
				height: var(--resizable-handler-size);
			}
			&__left,
			&__right {
				top: 0;
				bottom: 0;
				cursor: e-resize;
				width: var(--resizable-handler-size);
			}
		}
	}
</style>