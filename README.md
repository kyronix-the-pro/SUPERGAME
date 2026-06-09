<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>PeerJS Multiplayer 3D</title>

<style>
html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    font-family:Arial,sans-serif;
}

#ui{
    position:absolute;
    inset:0;
    background:#111;
    color:white;
    display:flex;
    justify-content:center;
    align-items:center;
    z-index:10;
}

#panel{
    background:#222;
    padding:20px;
    border-radius:12px;
    text-align:center;
}

input{
    width:250px;
    padding:10px;
    margin:5px;
}

button{
    padding:10px 20px;
    margin:5px;
}

#roomDisplay{
    position:absolute;
    top:10px;
    left:10px;
    color:white;
    background:rgba(0,0,0,.6);
    padding:10px;
    z-index:5;
}
</style>
</head>
<body>

<div id="ui">
<div id="panel">

<h2>PeerJS Multiplayer</h2>

<input id="username" placeholder="Username">

<br>

<button id="hostBtn">
Create Room
</button>

<br>

<input id="roomInput"
placeholder="Room ID">

<br>

<button id="joinBtn">
Join Room
</button>

</div>
</div>

<div id="roomDisplay"></div>

<script type="module">

import * as THREE
from "https://cdn.jsdelivr.net/npm/three@0.164.1/build/three.module.js";

import Peer
from "https://esm.sh/peerjs@1.5.4";

let username="";
let peer;
let roomId="";
let isHost=false;

const connections=[];

const remotePlayers={};

let scene;
let camera;
let renderer;
let localPlayer;

const roomDisplay=
document.getElementById(
"roomDisplay"
);

function randomColor(){
    return Math.floor(
        Math.random()*16777215
    );
}

function createCharacter(color){

    const g=
    new THREE.Group();

    const mat=
    new THREE.MeshStandardMaterial({
        color
    });

    const body=
    new THREE.Mesh(
        new THREE.BoxGeometry(
            1,
            1.5,
            .7
        ),
        mat
    );

    body.position.y=1.75;
    g.add(body);

    const head=
    new THREE.Mesh(
        new THREE.BoxGeometry(
            .8,.8,.8
        ),
        new THREE.MeshStandardMaterial({
            color:0xffddaa
        })
    );

    head.position.y=3;
    g.add(head);

    return g;
}

function nameTag(text){

    const c=
    document.createElement(
        "canvas"
    );

    c.width=256;
    c.height=64;

    const ctx=
    c.getContext("2d");

    ctx.fillStyle="white";
    ctx.font="30px Arial";
    ctx.textAlign="center";

    ctx.fillText(
        text,
        128,
        40
    );

    const sprite=
    new THREE.Sprite(
        new THREE.SpriteMaterial({
            map:new THREE.CanvasTexture(c)
        })
    );

    sprite.scale.set(
        4,
        1,
        1
    );

    sprite.position.y=4.5;

    return sprite;
}

function startWorld(){

    scene=
    new THREE.Scene();

    scene.background=
    new THREE.Color(
        0x87ceeb
    );

    camera=
    new THREE.PerspectiveCamera(
        75,
        innerWidth/innerHeight,
        .1,
        1000
    );

    renderer=
    new THREE.WebGLRenderer({
        antialias:true
    });

    renderer.setSize(
        innerWidth,
        innerHeight
    );

    document.body.appendChild(
        renderer.domElement
    );

    const light=
    new THREE.HemisphereLight(
        0xffffff,
        0x444444,
        2
    );

    scene.add(light);

    const floor=
    new THREE.Mesh(
        new THREE.BoxGeometry(
            250,
            1,
            250
        ),
        new THREE.MeshStandardMaterial({
            color:0x44aa44
        })
    );

    floor.position.y=-.5;

    scene.add(floor);

    localPlayer=
    createCharacter(
        randomColor()
    );

    localPlayer.add(
        nameTag(username)
    );

    scene.add(
        localPlayer
    );

    animate();
}

function updateRemote(data){

    if(!remotePlayers[data.id]){

        const p=
        createCharacter(
            0x0066ff
        );

        p.add(
            nameTag(
                data.username
            )
        );

        scene.add(p);

        remotePlayers[data.id]=p;
    }

    remotePlayers[data.id]
    .position.set(
        data.x,
        data.y,
        data.z
    );
}

function broadcast(data){

    connections.forEach(
        c=>{
            if(c.open){
                c.send(data);
            }
        }
    );
}

function setupConnection(conn){

    connections.push(conn);

    conn.on(
        "data",
        data=>{

            updateRemote(data);

            if(isHost){

                broadcast(data);
            }
        }
    );
}

document
.getElementById("hostBtn")
.onclick=()=>{

    username=
    document
    .getElementById("username")
    .value
    .trim();

    if(!username)return;

    peer=
    new Peer();

    peer.on(
        "open",
        id=>{

            roomId=id;

            roomDisplay.innerHTML=
            "Room ID: <b>"+id+"</b>";

            isHost=true;

            document
            .getElementById("ui")
            .style.display="none";

            startWorld();
        }
    );

    peer.on(
        "connection",
        conn=>{

            setupConnection(conn);
        }
    );
};

document
.getElementById("joinBtn")
.onclick=()=>{

    username=
    document
    .getElementById("username")
    .value
    .trim();

    roomId=
    document
    .getElementById("roomInput")
    .value
    .trim();

    if(!username)return;
    if(!roomId)return;

    peer=
    new Peer();

    peer.on(
        "open",
        ()=>{

            const conn=
            peer.connect(roomId);

            conn.on(
                "open",
                ()=>{

                    setupConnection(
                        conn
                    );

                    document
                    .getElementById(
                        "ui"
                    )
                    .style.display=
                    "none";

                    startWorld();
                }
            );
        }
    );
};

const keys={};

addEventListener(
"keydown",
e=>{
keys[e.key.toLowerCase()]=true;
});

addEventListener(
"keyup",
e=>{
keys[e.key.toLowerCase()]=false;
});

let vy=0;
let grounded=true;

function animate(){

    requestAnimationFrame(
        animate
    );

    const speed=.15;

    if(keys.w)
        localPlayer.position.z-=speed;

    if(keys.s)
        localPlayer.position.z+=speed;

    if(keys.a)
        localPlayer.position.x-=speed;

    if(keys.d)
        localPlayer.position.x+=speed;

    if(keys[" "]&&grounded){

        vy=.22;
        grounded=false;
    }

    vy-=.01;

    localPlayer.position.y+=vy;

    if(localPlayer.position.y<0){

        localPlayer.position.y=0;
        vy=0;
        grounded=true;
    }

    camera.position.set(
        localPlayer.position.x,
        localPlayer.position.y+6,
        localPlayer.position.z+8
    );

    camera.lookAt(
        localPlayer.position.x,
        localPlayer.position.y+2,
        localPlayer.position.z
    );

    if(peer){

        const packet={

            id:peer.id,

            username,

            x:localPlayer.position.x,

            y:localPlayer.position.y,

            z:localPlayer.position.z
        };

        broadcast(packet);
    }

    renderer.render(
        scene,
        camera
    );
}

addEventListener(
"resize",
()=>{

camera.aspect=
innerWidth/innerHeight;

camera.updateProjectionMatrix();

renderer.setSize(
innerWidth,
innerHeight
);
});
</script>

</body>
</html>
