from pathlib import Path

html = r"""<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Kart Arena Prototype</title>
<style>
html,body{margin:0;height:100%;overflow:hidden;background:#87ceeb;font-family:Arial}
#hud{position:absolute;top:10px;left:10px;background:rgba(0,0,0,.5);color:#fff;padding:8px;border-radius:6px}
</style>
</head>
<body>
<div id="hud">WASD/Arrows = Drive<br>Space = Boost</div>
<script type="module">
import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.164.1/build/three.module.js";

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x87ceeb);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,1000);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x444444,2));

const floor=new THREE.Mesh(
 new THREE.PlaneGeometry(200,200),
 new THREE.MeshStandardMaterial({color:0x55aa55})
);
floor.rotation.x=-Math.PI/2;
scene.add(floor);

for(let i=0;i<40;i++){
 const box=new THREE.Mesh(
  new THREE.BoxGeometry(2,2,2),
  new THREE.MeshStandardMaterial({color:Math.random()*0xffffff})
 );
 box.position.set((Math.random()-0.5)*180,1,(Math.random()-0.5)*180);
 scene.add(box);
}

const kart=new THREE.Group();
const body=new THREE.Mesh(
 new THREE.BoxGeometry(2,0.8,3),
 new THREE.MeshStandardMaterial({color:0xff3333})
);
kart.add(body);
scene.add(kart);

const keys={};
addEventListener("keydown",e=>keys[e.key.toLowerCase()]=true);
addEventListener("keyup",e=>keys[e.key.toLowerCase()]=false);

let speed=0;
let angle=0;

function loop(){
 requestAnimationFrame(loop);

 const accel=(keys.w||keys.arrowup)?0.01:0;
 const brake=(keys.s||keys.arrowdown)?0.01:0;

 speed+=accel;
 speed-=brake;
 speed*=0.98;

 const boost=keys[" "]?1.8:1;

 if(keys.a||keys.arrowleft) angle+=0.03*Math.sign(speed||1);
 if(keys.d||keys.arrowright) angle-=0.03*Math.sign(speed||1);

 kart.rotation.y=angle;
 kart.position.x-=Math.sin(angle)*speed*boost;
 kart.position.z-=Math.cos(angle)*speed*boost;

 camera.position.set(
   kart.position.x+Math.sin(angle)*8,
   6,
   kart.position.z+Math.cos(angle)*8
 );
 camera.lookAt(kart.position);

 renderer.render(scene,camera);
}
loop();

addEventListener("resize",()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>"""

path = "/mnt/data/kart_arena_prototype.html"
Path(path).write_text(html, encoding="utf-8")
print(path)
