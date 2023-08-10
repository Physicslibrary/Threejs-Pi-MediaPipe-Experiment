# Threejs-Pi-MediaPipe-Experiment
Exploring hand tracking with Luxonis' Oak-D on a Raspberry Pi and Threejs

# Experiment 1</br>

https://github.com/Physicslibrary/Threejs-Pi-MediaPipe-Experiment/assets/47191469/f67240aa-f5b8-43f8-9a49-eae79a70221f

Luxonis' webpage is a starting point for Threejs mediapipe hand tracking experiment.<br>

https://docs.luxonis.com/en/latest/#example-use-cases

Specifically, hand tracking by Geaxgx.</br>

https://github.com/geaxgx/depthai_hand_tracker

# Hardware
Raspberry Pi 4 Model B (4GB version)</br>
Luxonis' Oak-D</br>

# Software
Raspberry Pi OS (64-bit) with desktop</br>
Release date: May 3rd 2023</br>
Kernel version: 6.1</br>
Debian version: 11 (bullseye)</br>
Size: 818MB</br>

Three.js (r155)</br>

websocketd (pipe Linux stdin/stdout to javascript websocket)</br>

# Setup

https://docs.luxonis.com/projects/api/en/latest/install/?highlight=raspberry%20pi#raspberry-pi-os

<pre>
sudo curl -fL https://docs.luxonis.com/install_dependencies.sh | bash
</pre>

Download code from https://github.com/geaxgx/depthai_hand_tracker.</br>

# Prototyping

Open "HandTrackerRenderer.py" (eg. with Pi text editor Geany).</br>

Add "from sys import stdout" to line 3. Need this for websocketd to work.</br>

Add next 4 lines to line 139 (eg. after cv2.putText() in "if self.show_xyz:").</br>

<pre>
print(hand.landmarks[4,0],hand.landmarks[4,1],hand.landmarks[8,0],hand.landmarks[8,1],round(hand.xyz[0]),round(hand.xyz[1]),round(hand.xyz[2]))
stdout.flush()
cv2.circle(self.frame, (hand.landmarks[4][0], hand.landmarks[4][1]), 20, (0,255,0), -1)
cv2.circle(self.frame, (hand.landmarks[8][0], hand.landmarks[8][1]), 20, (0,255,0), -1)
</pre>

Four lines of python to process stdout before piping to websocketd to javascript "var ws = new WebSocket()".</br>

<pre>
  for line in sys.stdin:

	if(len(line)<40):
		print(line.rstrip())
		sys.stdout.flush()
</pre>

```
<!DOCTYPE html>
<html lang="en">
	<head>
		<title>threejs mediapipe hand tracking</title>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
		<link type="text/css" rel="stylesheet" href="main.css">
	</head>
	<body>

		<div id="info">
			<a href="https://threejs.org" target="_blank" rel="noopener"></a>threejs depthai mediapipe hand tracking<br/>
		</div>

		<!-- Import maps polyfill -->
		<!-- Remove this when import maps will be widely supported -->
		<script async src="https://unpkg.com/es-module-shims@1.6.3/dist/es-module-shims.js"></script>

		<script type="importmap">
			{
				"imports": {
					"three": "../build/three.module.js",
					"three/addons/": "./jsm/"
				}
			}
		</script>

		<script type="module">

import * as THREE from 'three';
import { OrbitControls } from './jsm/controls/OrbitControls.js';

var camera, scene, renderer, controls;
var geometry, thumb-tip, index-tip;

var text=[560,300,560,300,0,0,1000];

init();
animate();

function init() {

	var ws = new WebSocket('wss://127.0.0.1:8000');

	ws.onopen = function() {
	console.log("websocket onopen");
	};

	ws.onclose = function() {
	console.log("websocket onclose");
	};
 
	ws.onmessage = function(event) {
		text = event.data.split(' ');
//		console.log(event.data); // for debugging
	};
		
	scene = new THREE.Scene();
	scene.background = new THREE.Color(0x404040);

	var color = new THREE.Color(0x00ff00);
	var floor = new THREE.GridHelper(10,10,color,color);
	scene.add(floor);
	
	camera = new THREE.PerspectiveCamera(50, window.innerWidth/window.innerHeight, 0.1, 100);
	camera.position.set( 0.0, 2.0, 8.0 );
	
	renderer = new THREE.WebGLRenderer({antialias: true});
	renderer.setPixelRatio(window.devicePixelRatio );
	renderer.setSize(window.innerWidth, window.innerHeight);	
	renderer.outputColorSpace = THREE.SRGBColorSpace;
	document.body.appendChild(renderer.domElement);

	window.addEventListener( 'resize', onWindowResize );

	controls = new OrbitControls(camera, renderer.domElement);

	geometry = new THREE.BoxGeometry(0.4,0.4,0.4);

	thumb-tip = new THREE.Mesh(geometry, new THREE.MeshBasicMaterial({color: 0x00ff00}));
	thumb-tip.material.wireframe = true;
	scene.add(thumb-tip);

	index-tip = new THREE.Mesh(geometry, new THREE.MeshBasicMaterial({color: 0x00ff00}));
	index-tip.material.wireframe = true;
	scene.add(index-tip);
	
	}

	function onWindowResize() {

		camera.aspect = window.innerWidth / window.innerHeight;
		camera.updateProjectionMatrix();

		renderer.setSize( window.innerWidth, window.innerHeight );

	}
			
	function animate() {

		requestAnimationFrame(animate);
		controls.update();

		thumb-tip.position.x = (560-text[0])/50;
		thumb-tip.position.y = (300-text[1])/50;
		thumb-tip.position.z = (text[6]-1000)/50;

		index-tip.position.x = (560-text[2])/50;
		index-tip.position.y = (300-text[3])/50;
		index-tip.position.z = (text[6]-1000)/50;
		
		renderer.render(scene, camera);

	}

</script>
</body>
</html>
```

# References</br>

https://docs.luxonis.com/en/latest/#example-use-cases

https://github.com/geaxgx/depthai_hand_tracker

https://ai.googleblog.com/2019/08/on-device-real-time-hand-tracking-with.html

https://www.raspberrypi.com/software/operating-systems/

https://docs.luxonis.com/projects/api/en/latest/install/?highlight=raspberry%20pi#raspberry-pi-os

https://threejs.org/

An ARM websocketd 0.3.0 binary is available for Pi from.</b>

https://github.com/joewalnes/websocketd (should be a link to websocketd.com which has a download)





