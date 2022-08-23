# Graph Image Generator

Use Vue3 to build this Website, and use Scss as css preprocessor, and use Vite as build tool.

## Website

Click to see this [Website](https://graph-image-generator.vercel.app/).

<hr>

# Introduction

### Project Setup

```sh
npm install
```

#### Compile and Hot-Reload for Development

```sh
npm run dev
```

#### Compile and Minify for Production

```sh
npm run build
```

<br>

### Method

Use the following two methods to get the size of the canvas and the binary data of the graph.

```javascript
// Use this function to convert hex data to binary.
const convertBinary = (text, sizeX) => {
	// get hex number, and convert to binary, and then fill string with 0
	return parseInt(text || 0, 16).toString(2).padStart(sizeX, "0");
};
```

```javascript
// Use this function to get canvas X direction size & graph's frames.
const convertTextToData = (text) => {
	// get x direction size
	let dataWidth = text.split(":")[0].split(",")[0];
	// get frames
	let graphFrameDatas = text.split(":")[1].split("#")
		.map((text) =>
			text.split("|").map((frameRow) => convertBinary(frameRow, dataWidth)));
	return [dataWidth, graphFrameDatas];
};
```

For example:

```javascript
// Copy the result from "Graph Data Result" as data's value.
const dataText =
	"15,15:|||410|220|7f0|dd8|1ffc|17f4|410|360||||#|||410|1224|17f4|1ddc|ff8|410|808|||||";

// Use funtion to get Size and Frame's data.
convertTextToData(dataText); // expected to get: Array(2) ["15",[["000000000000000", ...],["000000000000000", ...]]]
```
<br>

### Add new frame & draw

- Click "Add new Graph" button to add a new canvas for new frame.

- Click and drag the mouse on the canvas to start drawing, then release the mouse to finish drawing.

- Uncheck "Mirror Mode" button to cancel mirror mode.

<img src="https://github.com/andy820621/Graph-Image-Generator/blob/main/gif/addmode%26draw.gif" width="600">

<br>

### Two ways to change different Frame

- Click "number" button to change different frame.

- Type the number key to quickly switch to that number's frame .

<img src="https://github.com/andy820621/Graph-Image-Generator/blob/main/gif/modeChange.gif" width="600">

<br>

### Two ways to start a new graph

- Click "clear" button to clear all frame.

- Change width or height value to get a different size canvas to start new Graph.

<img src="https://github.com/andy820621/Graph-Image-Generator/blob/main/gif/clear%26sizeChange.gif" width="600">

<br>

### Load Data

Click "loadData" button and enter data to show the Graph.

<img src="https://github.com/andy820621/Graph-Image-Generator/blob/main/gif/loadData.gif" width="600">
