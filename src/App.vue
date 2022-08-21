<script setup>
import { computed, reactive } from "vue";
import chunk from "lodash/chunk.js";

const data = reactive({
	graphModes: [],
	settings: {
		width: 15,
		height: 15,
	},
	mirrorModes: true,
	currentModeId: 0,
	editing: false,
});

init();
load(
	"15,15:|||410|220|7f0|dd8|1ffc|17f4|410|360||||#|||410|1224|17f4|1ddc|ff8|410|808|||||"
);
function init() {
	data.currentModeId = 0;
	data.graphModes = [];
	addMode();
}
function load(graphDatas) {
	let settings = graphDatas.split(":")[0].split(",");
	let mainData = graphDatas.split(":")[1];
	data.settings.width = parseInt(settings[0]);
	data.settings.height = parseInt(settings[1]);
	init();
	data.graphModes = mainData
		.split("#")
		.map((mode) =>
			mode
				.split("|")
				.map((mode) => convertBinary(mode))
				.join("")
		)
		.map((mode) =>
			mode.split("").map((text) => {
				return { value: +text };
			})
		);
}
function addMode() {
	data.graphModes.push(
		Array.from({ length: data.settings.width * data.settings.height }, () => {
			return { value: 0 };
		})
	);
	data.currentModeId = data.graphModes.length - 1;
}
function getArrayId(x, y) {
	return y * data.settings.width + x;
}
function toggleBlock(x, y, black) {
	graphData.value[getArrayId(x, y)].value = black
		? 1
		: +!graphData.value[getArrayId(x, y)].value;
	if (data.mirrorModes) {
		let mid = Math.floor(data.settings.width / 2);
		let nextX = mid - (x - mid);
		if (nextX !== x)
			graphData.value[getArrayId(nextX, y)].value = black
				? 1
				: +!graphData.value[getArrayId(nextX, y)].value;
	}
}
function removeMode(i) {
	if (data.currentModeId >= i) data.currentModeId -= 1;
	data.graphModes.splice(i, 1);
}
function convertHex(text) {
	return parseInt(text, 2).toString(16);
}
function convertBinary(text) {
	if (!text) text = 0;
	return parseInt(text, 16).toString(2).padStart(data.settings.width, 0);
}
function loadDataHandler() {
	load(prompt("Please enter Code."));
}

// computed
const graphData = computed(() => data.graphModes[data.currentModeId]);
const code = computed(() => {
	let result = `${data.settings.width},${data.settings.height}:`;

	result += data.graphModes
		.map((modeArray) =>
			chunk(
				modeArray.map((mode) => mode.value),
				data.settings.width
			)
				.map((line) => line.join(""))
				.map((line) => (convertHex(line) !== "0" ? convertHex(line) : ""))
				.join("|")
		)
		.join("#");

	return result;
});

// EventListener
window.addEventListener("keydown", (e) => {
	if (parseInt(e.key) > data.graphModes.length || isNaN(parseInt(e.key)))
		return;
	data.currentModeId = parseInt(e.key) - 1;
});
</script>

<template>
	<h1>Graph Data Generator</h1>

	<a
		href="https://github.com/andy820621/Graph-Image-Generator"
		target="_blank"
		class="github-corner"
		aria-label="View source on GitHub"
		><svg
			width="80"
			height="80"
			viewBox="0 0 250 250"
			style="
				fill: #fff;
				color: #282f38;
				position: absolute;
				top: 0;
				border: 0;
				right: 0;
			"
			aria-hidden="true"
		>
			<path
				d="M0,0 L115,115 L130,115 L142,142 L250,250 L250,0 Z"
				fill="hsl(200, 40%, 45%)"
			></path>
			<path
				d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2"
				fill="currentColor"
				style="transform-origin: 130px 106px"
				class="octo-arm"
			></path>
			<path
				d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z"
				fill="currentColor"
				class="octo-body"
			></path></svg
	></a>
	<main class="row">
		<div class="main">
			<label class="mirror">
				<input type="checkbox" v-model="data.mirrorModes" />
				<span>Mirorr Mode</span>
			</label>
			<!-- <div>
				<label for="editing">{{ data.editing }}</label>
			</div> -->

			<div
				class="graph"
				@mousedown="data.editing = true"
				@mouseup="data.editing = false"
				@mouseleave="data.editing = false"
			>
				<div class="row" v-for="y in data.settings.height" :key="'y' + y">
					<div
						class="block"
						:class="{
							active: graphData[getArrayId(x - 1, y - 1)]?.value === 1,
						}"
						v-for="x in data.settings.width"
						@click="toggleBlock(x - 1, y - 1)"
						@mousemove="data.editing ? toggleBlock(x - 1, y - 1, true) : ''"
						:key="'x' + x"
					>
						<!-- <span> {{ y }}-{{ x }} </span> -->
						<!-- <span>{{ graphData[getArrayId(x - 1, y - 1)].value }}</span> -->
					</div>
				</div>
			</div>

			<div class="modeTabs-container">
				<ul class="modeTabs">
					<li
						v-for="(mode, i) in data.graphModes"
						:key="i"
						:class="{ active: data.currentModeId === i }"
						@click="data.currentModeId = i"
					>
						{{ i + 1 }}
						<button @click.stop="removeMode(i)">x</button>
					</li>
				</ul>
				<button id="addMode" @click="addMode">Add new Graph</button>
			</div>
		</div>

		<div class="control">
			<h3>Control</h3>
			<div class="input-control">
				<label>Width</label>
				<input
					type="number"
					v-model.number="data.settings.width"
					@change="init"
				/>
			</div>
			<div class="input-control">
				<label>Height</label>
				<input
					type="number"
					v-model.number="data.settings.height"
					@change="init"
				/>
			</div>
			<div class="btn-control">
				<button @click="init">Clear</button>
				<button @click="loadDataHandler">Load Data</button>
			</div>

			<div class="textarea-control">
				<label for="code">✻ Graph Data Result:</label>
				<textarea name="code" :value="code" cols="24" rows="8"></textarea>
			</div>
		</div>
	</main>
</template>

<style lang="scss" scoped>
@mixin size($width, $height: $width) {
	width: $width;
	height: $height;
}

h1 {
	text-align: center;
}
button,
.modeTabs li {
	padding: 0.3rem 0.8rem;
	cursor: pointer;
	transition: 0.3s;
	&:active {
		transform: translateY(0.24rem);
	}
	&:hover {
		background-color: rgb(186, 199, 207);
	}
}
.row {
	display: flex;
}
main.row {
	gap: 2.4rem;
	@media (max-width: 768px) {
		flex-direction: column-reverse;
	}
	label.mirror {
		padding: 0;
		cursor: pointer;
		user-select: none;
		input[type="checkbox"] {
			opacity: 0;
			visibility: 0;
			width: 0;
			margin: 0;
		}
		input[type="checkbox"] + span {
			display: inline-block;
			padding-left: 1.8rem;
			line-height: 1.2rem;
			background: url(https://i.imgur.com/bZM5Itd.png) no-repeat left top;
			user-select: none;
			transition: 0.2s;
			&:hover {
				opacity: 0.8;
			}
		}
		input[type="checkbox"]:checked + span {
			background: url(https://i.imgur.com/JWm4WKA.png) no-repeat left top;
			opacity: 0.75;
		}
	}

	.main,
	.control {
		margin-top: 2.4rem;
		padding: 10px;
		display: flex;
		flex-direction: column;
		gap: 1rem;
		h3 {
			font-size: 1.5rem;
			margin: 0;
			letter-spacing: 0.08rem;
		}
		input,
		textarea,
		button {
			font-size: 1rem;
			padding: 0.24rem 0.5rem;
			border: none;
			border-radius: 0.24rem;
			max-width: 80vmin;
		}
		.input-control,
		.btn-control,
		.textarea-control {
			display: flex;
		}
		.input-control {
			gap: 1rem;
		}
		.btn-control {
			gap: 2rem;
			margin-top: 0.24rem;
		}
		.textarea-control {
			gap: 0.7rem;
			font-size: 1.2rem;
			flex-direction: column;
			margin-top: 1rem;
		}
	}
}
.graph {
	.block {
		@include size(5vmin);
		@media (min-width: 768px) {
			@include size(3.5vmin);
		}
		border: 1px solid #ddd;
		background-color: #eee;
		color: #333;
		cursor: pointer;
		font-size: 14px;
		&:hover {
			background-color: #ccc;
		}
		&.active {
			background-color: #333;
		}
	}
}

.modeTabs-container {
	button#addMode {
		width: max-content;
		padding: 0.6rem 1.3rem;
		@media (max-width: 768px) {
			margin: 1rem auto;
			display: block;
		}
	}
	ul {
		margin: 1rem auto;
		padding: 2rem 0;
		@media (min-width: 768px) {
			padding: 0;
			margin: 2.4rem 0;
		}
		list-style: none;
		display: flex;
		flex-wrap: wrap;
		column-gap: 1.25rem;
		row-gap: 1.6rem;
		width: 80%;
		@media (min-width: 768px) {
			width: 60vmin;
		}
		li {
			position: relative;
			background-color: #eee;
			color: #333;
			border-radius: 0.24rem;
			padding: 0.5rem 2rem;
			&.active {
				background-color: hsl(200, 40%, 45%);
				color: #eee;
			}
			button {
				position: absolute;
				width: auto;
				right: 0;
				top: 0;
				transform: translate(50%, -50%);
				background-color: rgb(251, 158, 107);
				&:hover {
					border-radius: 50%;
				}
			}
		}
	}
}

.github-corner:hover .octo-arm {
	animation: octocat-wave 560ms ease-in-out;
}
@keyframes octocat-wave {
	0%,
	100% {
		transform: rotate(0);
	}
	20%,
	60% {
		transform: rotate(-25deg);
	}
	40%,
	80% {
		transform: rotate(10deg);
	}
}
@media (max-width: 500px) {
	.github-corner:hover .octo-arm {
		animation: none;
	}
	.github-corner .octo-arm {
		animation: octocat-wave 560ms ease-in-out;
	}
}
</style>
