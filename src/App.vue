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
	<main class="row">
		<div class="main">
			<label class="mirror">
				<input type="checkbox" v-model="data.mirrorModes" />
				<span>Mirorr</span>
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

			<div>
				<textarea name="code" :value="code" cols="24" rows="8"></textarea>
			</div>

			<div class="btn-control">
				<button @click="init">Clear</button>
				<button @click="loadDataHandler">Load Data</button>
			</div>
		</div>
	</main>
</template>

<style lang="scss" scoped>
@mixin size($width, $height: $width) {
	width: $width;
	height: $height;
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
		padding: 10px;
		display: flex;
		flex-direction: column;
		gap: 1rem;
		input,
		textarea,
		button {
			font-size: 1rem;
			padding: 0.24rem 0.5rem;
			border: none;
			border-radius: 0.24rem;
		}
		.input-control,
		.btn-control {
			display: flex;
			gap: 1rem;
		}
	}
}
.graph {
	.block {
		@include size(2.4vmin);
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
	}
	ul {
		padding: 0;
		margin: 2.4rem 0;
		list-style: none;
		display: flex;
		flex-wrap: wrap;
		column-gap: 1.25rem;
		row-gap: 1.6rem;
		width: 36vmin;
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
</style>
