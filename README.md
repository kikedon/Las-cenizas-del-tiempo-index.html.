# Las-cenizas-del-tiempo-index.html.
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Las Cenizas del Olimpo</title>

<style>
*{box-sizing:border-box;margin:0;padding:0}
body{overflow:hidden;background:#080706;color:white;font-family:Georgia,serif}
canvas{display:block}
#menu,#ending{
 position:fixed;inset:0;z-index:20;
 display:flex;flex-direction:column;
 align-items:center;justify-content:center;
 text-align:center;padding:25px;
 background:radial-gradient(ellipse,#54301e,#110c0a 65%,#050404);
}
h1{font-size:clamp(30px,7vw,68px);color:#d9a36c;text-shadow:3px 4px 8px black;margin-bottom:20px}
.subtitle{color:#c9b29a;max-width:550px;line-height:1.7;margin-bottom:25px}
button{font-family:Georgia,serif;cursor:pointer}
.mainbtn{
 padding:16px 40px;background:#802b1b;
 color:white;border:2px solid #d6a36d;font-size:19px;
}
.mainbtn:active{background:#bd5230}
#hud{display:none;position:fixed;z-index:5;top:12px;left:12px;right:12px;pointer-events:none}
#healthbox{width:190px}
#healthbox small{color:#e7c9a1}
.bar{height:13px;background:#332a25;border:1px solid #a58b70;margin-top:4px}
#healthbar{height:100%;width:100%;background:#a52b20}
#bossbox{display:none;position:absolute;top:60px;left:50%;transform:translateX(-50%);width:min(350px,65vw);text-align:center}
#bossbar{height:12px;background:#9b2019;width:100%}
#chapter{position:absolute;right:5px;top:5px;color:#e5bd8d;text-align:right;font-size:14px}
#message{
 position:fixed;z-index:6;bottom:105px;left:50%;
 transform:translateX(-50%);width:90%;max-width:650px;
 text-align:center;color:#f5d8b0;font-size:17px;
 text-shadow:2px 2px 5px black;pointer-events:none;
}
#objective{
 position:fixed;z-index:5;top:85px;left:15px;
 font-size:12px;color:#e3c8a5;pointer-events:none;
}
#controls{
 display:none;position:fixed;bottom:15px;left:12px;right:12px;
 z-index:10;justify-content:space-between;align-items:end;
}
.pad,.actions{display:flex;gap:8px}
.control{
 width:57px;height:57px;border-radius:50%;
 color:white;background:#35251fdc;border:1px solid #c69b69;
 font-size:22px;touch-action:none;
}
.control:active{background:#9c4c2d}
#attack{width:68px;height:68px;background:#70251f}
#pause{
 position:fixed;right:15px;top:15px;z-index:12;
 display:none;background:#33241d;color:white;
 border:1px solid #b88c5a;padding:8px 13px;
}
#ending{display:none}
#ending p{max-width:550px;line-height:1.8;color:#e0c5a5;margin:15px}
@media(min-width:800px){
 #controls{max-width:650px;left:50%;transform:translateX(-50%)}
}
</style>
</head>

<body>

<div id="menu">
 <h1>LAS CENIZAS<br>DEL OLIMPO</h1>
 <p class="subtitle">
  Cuando los dioses abandonaron a los mortales,
  una antigua maldición despertó bajo las ruinas de Grecia.
  Un guerrero deberá descubrir la verdad que el Olimpo intenta ocultar.
 </p>
 <button class="mainbtn" id="start">COMENZAR AVENTURA</button>
 <p style="margin-top:25px;color:#a58d77;font-size:12px">
  WASD o flechas: mover · F: atacar · ESPACIO: saltar
 </p>
</div>

<div id="hud">
 <div id="healthbox">
  <small>VIDA DEL GUERRERO</small>
  <div class="bar"><div id="healthbar"></div></div>
 </div>
 <div id="chapter">CAPÍTULO I</div>
 <div id="bossbox">
  <small id="bossname">GUARDIÁN</small>
  <div class="bar"><div id="bossbar"></div></div>
 </div>
</div>

<div id="objective"></div>
<div id="message"></div>
<button id="pause">PAUSAR</button>

<div id="controls">
 <div class="pad">
  <button class="control" id="left">◀</button>
  <button class="control" id="right">▶</button>
 </div>
 <div class="actions">
  <button class="control" id="jump">▲</button>
  <button class="control" id="attack">⚔</button>
 </div>
</div>

<div id="ending">
 <h1>LAS CENIZAS<br>DEL OLIMPO</h1>
 <p id="endingtext">
  El destino de los dioses ha cambiado.
 </p>
 <button class="mainbtn" onclick="location.reload()">VOLVER A JUGAR</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>
"use strict";

// ==========================================
// CONFIGURACIÓN
// ==========================================

let scene,camera,renderer,clock;
let player,weapon;
let enemies=[];
let decorations=[];
let keys={};
let health=100;
let maxHealth=100;
let stage=0;
let playing=false;
let paused=false;
let gameEnded=false;
let grounded=true;
let verticalSpeed=0;
let attackCooldown=0;
let dodgeCooldown=0;
let invulnerable=0;
let messageTimer=0;
let wave=0;
let bossMode=false;
let bossHealth=0;
let bossMaxHealth=0;
let objective="";
let elapsed=0;

const stages=[
 {
  name:"CAPÍTULO I: EL TEMPLO OLVIDADO",
  objective:"Derrota a los guardianes del templo.",
  intro:"Las ruinas permanecen en silencio. Una voz antigua susurra tu nombre. Alguien ha despertado bajo estas piedras.",
  waves:[
   {count:3,hp:65},
   {count:4,hp:85}
  ],
  boss:"EL CENTINELA DE PIEDRA"
 },
 {
  name:"CAPÍTULO II: EL VALLE DE LOS CONDENADOS",
  objective:"Sobrevive a las criaturas del valle.",
  intro:"Tras cruzar el templo, descubres un valle cubierto por cenizas. Los espíritus de los antiguos soldados todavía protegen el camino.",
  waves:[
   {count:4,hp:100},
   {count:5,hp:115}
  ],
  boss:"EL VERDUGO DEL INFRAMUNDO"
 },
 {
  name:"CAPÍTULO III: LA PUERTA DEL OLIMPO",
  objective:"Enfréntate al último guardián.",
  intro:"Una gigantesca puerta se levanta ante ti. Detrás de ella se encuentra la verdad sobre la maldición.",
  waves:[
   {count:5,hp:130},
   {count:6,hp:145}
  ],
  boss:"EL TITÁN DE LAS CENIZAS"
 }
];

// ==========================================
// INICIAR MOTOR 3D
// ==========================================

function initialize(){

 scene=new THREE.Scene();
 scene.background=new THREE.Color(0x17100d);
 scene.fog=new THREE.Fog(0x17100d,18,55);

 camera=new THREE.PerspectiveCamera(
  65,window.innerWidth/window.innerHeight,0.1,150
 );

 renderer=new THREE.WebGLRenderer({
  antialias:false,
  powerPreference:"low-power"
 });

 renderer.setPixelRatio(Math.min(window.devicePixelRatio,1));
 renderer.setSize(window.innerWidth,window.innerHeight);
 renderer.shadowMap.enabled=false;

 document.body.appendChild(renderer.domElement);

 clock=new THREE.Clock();

 const ambient=new THREE.HemisphereLight(
  0xffd5ad,0x21130e,1.7
 );
 scene.add(ambient);

 const sunlight=new THREE.DirectionalLight(0xffc18b,1.5);
 sunlight.position.set(-8,15,8);
 scene.add(sunlight);

 createEnvironment();
 createPlayer();

 camera.position.set(0,5,9);

 window.addEventListener("resize",resize);

 animate();
}

// ==========================================
// ESCENARIO
// ==========================================

function createEnvironment(){

 const floor=new THREE.Mesh(
  new THREE.PlaneGeometry(100,100),
  new THREE.MeshStandardMaterial({color:0x49403a})
 );

 floor.rotation.x=-Math.PI/2;
 floor.position.y=-0.12;
 scene.add(floor);

 const stone=new THREE.MeshStandardMaterial({
  color:0x716052,roughness:1
 });

 const darkStone=new THREE.MeshStandardMaterial({
  color:0x39302b,roughness:1
 });

 // Columnas
 for(let i=0;i<12;i++){

  let side=i%2===0?-1:1;
  let z=-i*3-2;

  let pillar=new THREE.Mesh(
   new THREE.BoxGeometry(1.1,6,1.1),stone
  );

  pillar.position.set(side*6,3,z);
  scene.add(pillar);

  let top=new THREE.Mesh(
   new THREE.BoxGeometry(1.7,.4,1.7),darkStone
  );

  top.position.set(side*6,6.1,z);
  scene.add(top);
 }

 // Paredes
 for(let side of [-1,1]){
  let wall=new THREE.Mesh(
   new THREE.BoxGeometry(.7,5,70),darkStone
  );
  wall.position.set(side*9,2.5,-23);
  scene.add(wall);
 }

 // Piedras decorativas
 for(let i=0;i<35;i++){

  let rock=new THREE.Mesh(
   new THREE.DodecahedronGeometry(
    .3+Math.random()*.7,0
   ),
   stone
  );

  rock.position.set(
   (Math.random()-.5)*16,
   .1,
   -Math.random()*48
  );

  rock.rotation.set(
   Math.random(),Math.random(),Math.random()
  );

  scene.add(rock);
  decorations.push(rock);
 }

 // Antorchas
 for(let z of [-4,-15,-28,-42]){

  for(let x of [-7,7]){

   const torch=new THREE.Group();

   const base=new THREE.Mesh(
    new THREE.CylinderGeometry(.15,.25,1,6),
    darkStone
   );

   base.position.y=.5;
   torch.add(base);

   const flame=new THREE.Mesh(
    new THREE.IcosahedronGeometry(.32,0),
    new THREE.MeshBasicMaterial({color:0xff6a18})
   );

   flame.position.y=1.2;
   torch.add(flame);

   torch.position.set(x,0,z);
   scene.add(torch);

   const light=new THREE.PointLight(0xff7020,5,9);
   light.position.set(x,1.5,z);
   scene.add(light);
  }
 }

 // Puerta de piedra
 const gate=new THREE.Mesh(
  new THREE.BoxGeometry(9,9,1.3),
  new THREE.MeshStandardMaterial({color:0x55473e})
 );

 gate.position.set(0,4.5,-51);
 scene.add(gate);
}

// ==========================================
// PERSONAJE
// ==========================================

function createPlayer(){

 player=new THREE.Group();

 const skin=new THREE.MeshStandardMaterial({
  color:0xb38b72
 });

 const armor=new THREE.MeshStandardMaterial({
  color:0x8b2920,roughness:.9
 });

 const metal=new THREE.MeshStandardMaterial({
  color:0xb5a08a,metalness:.6,roughness:.5
 });

 const body=new THREE.Mesh(
  new THREE.BoxGeometry(.85,1.15,.5),armor
 );
 body.position.y=1.3;
 player.add(body);

 const head=new THREE.Mesh(
  new THREE.SphereGeometry(.32,8,7),skin
 );
 head.position.y=2.15;
 player.add(head);

 const leftArm=new THREE.Mesh(
  new THREE.BoxGeometry(.25,.9,.3),skin
 );
 leftArm.position.set(-.58,1.25,0);
 player.add(leftArm);

 const rightArm=new THREE.Mesh(
  new THREE.BoxGeometry(.25,.9,.3),skin
 );
 rightArm.position.set(.58,1.25,0);
 player.add(rightArm);

 const legMaterial=new THREE.MeshStandardMaterial({
  color:0x39302a
 });

 for(let x of [-.23,.23]){
  const leg=new THREE.Mesh(
   new THREE.BoxGeometry(.3,.85,.35),legMaterial
  );
  leg.position.set(x,.42,0);
  player.add(leg);
 }

 weapon=new THREE.Group();

 const blade=new THREE.Mesh(
  new THREE.BoxGeometry(.13,1.3,.12),metal
 );

 blade.position.y=.65;
 weapon.add(blade);

 const handle=new THREE.Mesh(
  new THREE.BoxGeometry(.25,.18,.22),
  new THREE.MeshStandardMaterial({color:0x58301d})
 );

 handle.position.y=.1;
 weapon.add(handle);

 weapon.position.set(.7,1.25,.1);
 weapon.rotation.z=-.6;
 player.add(weapon);

 player.position.set(0,0,2);
 scene.add(player);
}

// ==========================================
// CREAR ENEMIGOS
// ==========================================

function createEnemy(type="normal",hp=70){

 const enemy=new THREE.Group();

 const boss=type==="boss";

 const bodyMat=new THREE.MeshStandardMaterial({
  color:boss?0x49302c:0x393c42,
  roughness:1
 });

 const body=new THREE.Mesh(
  new THREE.BoxGeometry(
   boss?1.8:1,
   boss?2.5:1.7,
   boss?1.2:.7
  ),
  bodyMat
 );

 body.position.y=boss?1.35:1;
 enemy.add(body);

 const head=new THREE.Mesh(
  new THREE.SphereGeometry(boss?.55:.38,8,6),
  new THREE.MeshStandardMaterial({color:0x766158})
 );

 head.position.y=boss?2.9:2;
 enemy.add(head);

 const eyes=new THREE.Mesh(
  new THREE.BoxGeometry(boss?.7:.48,.13,.1),
  new THREE.MeshBasicMaterial({color:0xff1c0a})
 );

 eyes.position.set(0,boss?2.95:2,.38);
 enemy.add(eyes);

 // Brazos
 for(let side of [-1,1]){
  const arm=new THREE.Mesh(
   new THREE.BoxGeometry(boss?.5:.3,boss?1.7:1,.4),
   bodyMat
  );

  arm.position.set(side*(boss?1.1:.65),1,0);
  enemy.add(arm);
 }

 enemy.position.set(
  (Math.random()-.5)*6,0,-12-Math.random()*5
 );

 scene.add(enemy);

 const data={
  mesh:enemy,
  hp:hp,
  maxHp:hp,
  boss:boss,
  attackTime:0,
  dead:false,
  speed:boss?1.15:1.5
 };

 enemies.push(data);
 return data;
}

function spawnWave(){

 const info=stages[stage].waves[wave];

 for(let i=0;i<info.count;i++){
  createEnemy("normal",info.hp);
 }

 updateObjective();
}

function spawnBoss(){

 bossMode=true;

 let hp=stage===0?650:stage===1?900:1300;
 bossHealth=hp;
 bossMaxHealth=hp;

 const names=[
  "EL CENTINELA DE PIEDRA",
  "EL VERDUGO DEL INFRAMUNDO",
  "EL TITÁN DE LAS CENIZAS"
 ];

 document.getElementById("bossbox").style.display="block";
 document.getElementById("bossname").textContent=names[stage];

 const boss=createEnemy("boss",hp);
 boss.mesh.position.set(0,0,-10);
}

// ==========================================
// MENSAJES E INTERFAZ
// ==========================================

function say(text,duration=5){

 const el=document.getElementById("message");
 el.textContent=text;
 messageTimer=duration;
}

function updateObjective(){

 const remaining=enemies.filter(e=>!e.dead).length;

 objective=stages[stage].objective+
 " Enemigos restantes: "+remaining;

 document.getElementById("objective").textContent=objective;

}

function updateHealth(){

 document.getElementById("healthbar").style.width=
 Math.max(0,health/maxHealth*100)+"%";

 if(bossMode){
  document.getElementById("bossbar").style.width=
  Math.max(0,bossHealth/bossMaxHealth*100)+"%";
 }
}

// ==========================================
// COMBATE
// ==========================================

function attack(){

 if(!playing||paused||attackCooldown>0)return;

 attackCooldown=.65;

 weapon.rotation.z=-1.8;

 setTimeout(()=>{
  if(weapon)weapon.rotation.z=-.6;
 },220);

 let target=null;
 let best=3.3;

 for(const e of enemies){

  if(e.dead)continue;

  let distance=player.position.distanceTo(e.mesh.position);

  if(distance<best){
   best=distance;
   target=e;
  }
 }

 if(target){

  const damage=target.boss?65:45;
  target.hp-=damage;

  target.mesh.position.z+=.35;

  if(target.boss){
   bossHealth=Math.max(0,target.hp);
  }

  say("¡Golpe certero!",1.1);

  if(target.hp<=0){
   target.dead=true;
   scene.remove(target.mesh);

   if(target.boss){
    bossDefeated();
   }else{
    say("El guardián ha caído.",1.5);
    updateObjective();
   }
  }

  updateHealth();
 }
}

function takeDamage(amount){

 if(invulnerable>0||!playing)return;

 health-=amount;
 invulnerable=1.1;

 player.position.x+=
 (Math.random()>.5?1:-1)*.5;

 updateHealth();

 if(health<=0){
  gameOver();
 }
}

function dodge(){

 if(!playing||dodgeCooldown>0)return;

 dodgeCooldown=1.2;
 invulnerable=.55;

 player.position.x+=
 player.position.x>0?-1.3:1.3;

 player.position.x=THREE.MathUtils.clamp(
  player.position.x,-7,7
 );
}

// ==========================================
// INTELIGENCIA ENEMIGA
// ==========================================

function updateEnemies(dt){

 for(const e of enemies){

  if(e.dead)continue;

  const mesh=e.mesh;

  const dx=player.position.x-mesh.position.x;
  const dz=player.position.z-mesh.position.z;
  const distance=Math.sqrt(dx*dx+dz*dz);

  if(distance>1.9){

   const speed=e.speed*dt;

   mesh.position.x+=dx/(distance||1)*speed;
   mesh.position.z+=dz/(distance||1)*speed;

  }else{

   e.attackTime-=dt;

   if(e.attackTime<=0){
    e.attackTime=e.boss?1.5:2;
    takeDamage(e.boss?13:8);
    say("¡Cuidado! Te han atacado.",1);
   }
  }

  mesh.rotation.y=Math.atan2(dx,dz);
 }
}

// ==========================================
// PROGRESIÓN DE HISTORIA
// ==========================================

function checkProgress(){

 if(!playing||bossMode)return;

 const alive=enemies.filter(e=>!e.dead).length;

 if(alive===0){

  if(wave<stages[stage].waves.length-1){

   wave++;

   say("Has superado la oleada. Prepárate para lo que viene.",3);

   setTimeout(()=>{
    if(playing&&!paused){
     spawnWave();
    }
   },2200);

  }else{

   say("El verdadero guardián ha despertado...",4);

   setTimeout(()=>{
    if(playing&&!paused)spawnBoss();
   },3000);
  }
 }
}

function bossDefeated(){

 bossMode=false;
 document.getElementById("bossbox").style.display="none";

 if(stage===2){
  victory();
  return;
 }

 say("La puerta antigua se ha abierto. Continúa, guerrero.",4);

 setTimeout(()=>{
  if(!playing)return;
  nextStage();
 },4000);
}

function nextStage(){

 stage++;
 wave=0;
 bossMode=false;
 health=Math.min(maxHealth,health+35);

 for(const e of enemies){
  scene.remove(e.mesh);
 }

 enemies=[];

 player.position.set(0,0,2);

 document.getElementById("chapter").textContent=
 stages[stage].name;

 updateHealth();

 say(stages[stage].intro,7);

 document.getElementById("objective").textContent=
 stages[stage].objective;

 setTimeout(()=>{
  if(playing&&!paused){
   spawnWave();
  }
 },6500);
}

// ==========================================
// FINAL
// ==========================================

function victory(){

 playing=false;
 gameEnded=true;

 document.getElementById("hud").style.display="none";
 document.getElementById("controls").style.display="none";
 document.getElementById("objective").style.display="none";

 document.getElementById("endingtext").textContent=
 "El Titán de las Cenizas ha sido derrotado. "+
 "Las puertas del Olimpo se abren lentamente. "+
 "El guerrero descubre que la maldición no fue obra de los mortales, "+
 "sino de los mismos dioses que juraron protegerlos. "+
 "La verdad ha sido revelada. Pero la guerra apenas comienza. "+
 "FIN DE LA PRIMERA AVENTURA.";

 document.getElementById("ending").style.display="flex";
}

function gameOver(){

 playing=false;

 document.getElementById("ending").style.display="flex";
 document.getElementById("endingtext").textContent=
 "El guerrero ha caído. Las sombras vuelven a cubrir el templo. "+
 "El Olimpo todavía guarda sus secretos.";
}

// ==========================================
// MOVIMIENTO
// ==========================================

function movePlayer(dt){

 let direction=0;

 if(keys["ArrowLeft"]||keys["a"]||keys["A"])direction-=1;
 if(keys["ArrowRight"]||keys["d"]||keys["D"])direction+=1;

 if(direction!==0){

  player.position.x+=direction*5*dt;
  player.rotation.y=direction*.2;

 }

 player.position.x=THREE.MathUtils.clamp(
  player.position.x,-7,7
 );

 // Movimiento adelante y atrás
 if(keys["ArrowUp"]||keys["w"]||keys["W"]){
  player.position.z-=4*dt;
 }

 if(keys["ArrowDown"]||keys["s"]||keys["S"]){
  player.position.z+=4*dt;
 }

 player.position.z=THREE.MathUtils.clamp(
  player.position.z,-47,5
 );

 // Salto
 if(!grounded){
  verticalSpeed-=17*dt;
  player.position.y+=verticalSpeed*dt;

  if(player.position.y<=0){
   player.position.y=0;
   grounded=true;
   verticalSpeed=0;
  }
 }

}

// ==========================================
// CÁMARA
// ==========================================

function updateCamera(dt){

 const targetX=player.position.x;
 const targetZ=player.position.z+8;

 camera.position.x+=
 (targetX-camera.position.x)*Math.min(dt*4,1);

 camera.position.y+=
 (5.2-camera.position.y)*Math.min(dt*3,1);

 camera.position.z+=
 (targetZ-camera.position.z)*Math.min(dt*4,1);

 camera.lookAt(
  player.position.x,
  1.1+player.position.y,
  player.position.z-3
 );
}

// ==========================================
// ACTUALIZACIÓN DEL JUEGO
// ==========================================

function animate(){

 requestAnimationFrame(animate);

 const dt=Math.min(clock.getDelta(),.05);

 if(playing&&!paused){

  elapsed+=dt;

  attackCooldown=Math.max(0,attackCooldown-dt);
  dodgeCooldown=Math.max(0,dodgeCooldown-dt);
  invulnerable=Math.max(0,invulnerable-dt);

  if(messageTimer>0){
   messageTimer-=dt;
   if(messageTimer<=0){
    document.getElementById("message").textContent="";
   }
  }

  movePlayer(dt);
  updateEnemies(dt);
  updateCamera(dt);
  checkProgress();

  // Parpadeo de invulnerabilidad
  player.visible=invulnerable<=0||
   Math.floor(elapsed*12)%2===0;

  player.visible=true;
 }

 renderer.render(scene,camera);
}

// ==========================================
// CONTROLES
// ==========================================

document.addEventListener("keydown",e=>{

 keys[e.key]=true;

 if(e.key===" "){
  e.preventDefault();

  if(grounded&&playing&&!paused){
   grounded=false;
   verticalSpeed=7;
  }
 }

 if(e.key==="f"||e.key==="F"){
  attack();
 }

 if(e.key==="Shift"){
  dodge();
 }

});

document.addEventListener("keyup",e=>{
 keys[e.key]=false;
});

function bindHold(id,key){

 const el=document.getElementById(id);

 el.addEventListener("pointerdown",e=>{
  e.preventDefault();
  keys[key]=true;
 });

 ["pointerup","pointerleave","pointercancel"].forEach(event=>{
  el.addEventListener(event,()=>{
   keys[key]=false;
  });
 });
}

bindHold("left","ArrowLeft");
bindHold("right","ArrowRight");

document.getElementById("attack").addEventListener(
 "pointerdown",e=>{
  e.preventDefault();
  attack();
 }
);

document.getElementById("jump").addEventListener(
 "pointerdown",e=>{
  e.preventDefault();

  if(grounded&&playing&&!paused){
   grounded=false;
   verticalSpeed=7;
  }
 }
);

// ==========================================
// PAUSA Y COMIENZO
// ==========================================

document.getElementById("pause").onclick=()=>{

 if(!playing)return;

 paused=!paused;

 document.getElementById("pause").textContent=
 paused?"CONTINUAR":"PAUSAR";

 if(paused){
  say("Juego pausado",100);
 }else{
  say("Continúa tu aventura.",2);
 }
};

document.getElementById("start").onclick=()=>{

 if(typeof THREE==="undefined"){
  alert("No se pudo cargar el motor 3D. Comprueba tu conexión a internet.");
  return;
 }

 document.getElementById("menu").style.display="none";
 document.getElementById("hud").style.display="block";
 document.getElementById("controls").style.display="flex";
 document.getElementById("pause").style.display="block";

 initialize();

 playing=true;

 document.getElementById("chapter").textContent=
 stages[0].name;

 say(stages[0].intro,8);

 setTimeout(()=>{
  if(playing&&!paused){
   spawnWave();
  }
 },8500);
};

</script>
</body>
</html>