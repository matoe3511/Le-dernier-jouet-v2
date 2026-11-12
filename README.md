<!DOCTYPE html>
<html lang="fr"><head><meta charset="utf-8"><title>Le Dernier Jouet – 1re personne</title>
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<style>
:root{--bg:#050000;--fg:#ddd;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#050000;--fg:#ddd}}
:root[data-theme="dark"]{--bg:#050000;--fg:#ddd}
html,body{height:100%;margin:0;background:var(--bg);color:var(--fg);font-family:Arial,sans-serif}
body{display:flex;align-items:center;justify-content:center}
canvas{max-width:100%;max-height:100%;aspect-ratio:1000/650;cursor:crosshair;background:#000;touch-action:none;-webkit-user-select:none;user-select:none}
body{overflow:hidden}#rot{display:none;position:fixed;inset:0;background:#000;color:#ddd;align-items:center;justify-content:center;text-align:center;font-size:22px;z-index:9;padding:20px}
@media (orientation:portrait) and (pointer:coarse){#rot{display:flex}}
</style></head><body><div id="rot">📱 Tourne ton téléphone en mode paysage pour jouer</div><canvas id="c"></canvas>
<script>
const cv=document.getElementById('c'),W=1000,H=650,T=40,C=40,R=28;let X=cv.getContext('2d');
cv.width=W;cv.height=H;
const ACT=[['up','Avancer'],['down','Reculer'],['left','Pas à gauche'],['right','Pas à droite'],['run','Courir (maintenir)'],['crouch','S’accroupir (maintenir)'],['torch','Lampe torche'],['gps','GPS on/off'],['hide','Se cacher (carton)'],['battery','Changer la pile'],['interact','Ramasser / Interagir']];
const DEF={up:'z',down:'s',left:'q',right:'d',run:'shift',crouch:'c',torch:'f',gps:'g',hide:'h',battery:'b',interact:'e',bino:'v',bmode:'n'};
let K={...DEF};try{Object.assign(K,JSON.parse(localStorage.getItem('keys2')||'{}'))}catch(e){}
const saveK=()=>{try{localStorage.setItem('keys2',JSON.stringify(K))}catch(e){}};
const kn=k=>k===' '?'ESPACE':k.toUpperCase();
const rnd=(a,b)=>a+Math.random()*(b-a),hyp=Math.hypot;const SECRET='k';
/* ---------- SONS ---------- */
let AC,NB;
function ac(){if(!AC){AC=new(window.AudioContext||window.webkitAudioContext)();NB=AC.createBuffer(1,AC.sampleRate,AC.sampleRate);const d=NB.getChannelData(0);for(let i=0;i<d.length;i++)d[i]=Math.random()*2-1}AC.resume();return AC}
function osc(type,f0,f1,d,v,dl=0){try{const a=ac(),o=a.createOscillator(),g=a.createGain(),t=a.currentTime+dl;o.type=type;o.frequency.setValueAtTime(f0,t);o.frequency.exponentialRampToValueAtTime(f1,t+d);g.gain.setValueAtTime(v,t);g.gain.exponentialRampToValueAtTime(.0001,t+d);o.connect(g).connect(a.destination);o.start(t);o.stop(t+d)}catch(e){}}
function nz(d,v,fl,q,ty){try{const a=ac(),s=a.createBufferSource(),f=a.createBiquadFilter(),g=a.createGain(),t=a.currentTime;s.buffer=NB;s.loop=true;f.type=ty||'lowpass';f.frequency.value=fl;f.Q.value=q||1;g.gain.setValueAtTime(v,t);g.gain.exponentialRampToValueAtTime(.0001,t+d);s.connect(f).connect(g).connect(a.destination);s.start(t);s.stop(t+d)}catch(e){}}
const S={heart:v=>{osc('sine',60,30,.15,v);osc('sine',50,28,.15,v*.7,.2)},
scare:k=>{osc('sine',55,22,1.1,.9);nz(.15,.8,8000,1,'highpass');const v={lytrho:0,collector:1}[k]??Math.floor(Math.random()*4);
if(v==0){osc('sawtooth',1100,80,.9,.5);nz(.9,.6,3000)}
else if(v==1){S.scream();nz(.8,.6,4000);osc('sawtooth',300,60,.9,.4)}
else if(v==2){S.bang();setTimeout(S.scream,80)}
else{osc('square',60,30,1,.6);osc('sawtooth',2000,300,.6,.3);nz(.8,.5,1500,1,'highpass')}},
bang:()=>{nz(.35,.9,250);osc('sine',90,30,.5,.8)},
laugh:()=>{for(let i=0;i<4;i++)osc('sine',900+i*40,600,.14,.12,i*.2)},
box:()=>{[659,587,523,587,659,659,659].forEach((f,i)=>osc('sine',f,f,.5,.1,i*.4))},
glass:()=>{nz(.6,.5,6000,1,'highpass');osc('sine',3200,2500,.3,.15)},
scream:()=>{osc('sawtooth',600,1500,.7,.35);osc('square',620,1480,.7,.15)},
breath:()=>{nz(.7,.12,900,2,'bandpass');setTimeout(()=>nz(.7,.1,800,2,'bandpass'),900)},
steps:()=>{for(let i=0;i<6;i++)setTimeout(()=>nz(.08,.14,350),i*350)},
scratch:()=>nz(1,.15,5000,3,'bandpass'),
growl:()=>{osc('sawtooth',70,40,1.1,.35);nz(1.1,.3,200)},
shout:()=>{osc('sawtooth',200,330,.7,.25);osc('sawtooth',212,340,.7,.2)},
whisper:()=>nz(1.6,.18,3000,2,'bandpass'),creak:()=>osc('sawtooth',160,90,.9,.1),
click:()=>nz(.04,.3,3000),step:()=>nz(.08,.2,400),
pickup:()=>{osc('sine',880,880,.6,.2);osc('sine',1320,1320,.6,.15)}};
let droneOn=false;
function drone(){if(droneOn)return;droneOn=true;try{const a=ac(),g=a.createGain();g.gain.value=.07;g.connect(a.destination);[40,42,63].forEach(f=>{const o=a.createOscillator();o.frequency.value=f;o.connect(g);o.start()})}catch(e){}}
/* ---------- CARTE ---------- */
let ROOMS,CORR,g,MAP;
const MAPS=[{name:'Le Manoir',rooms:{A:[1,1,6,6],B:[10,1,8,6],C:[22,1,8,6],D:[33,1,6,8],E:[1,10,8,7],F:[12,10,10,7],G:[25,11,12,6],H:[1,19,10,7],I:[14,19,8,7],J:[25,19,12,7]},corr:[[7,3,3,2],[18,3,4,2],[30,3,3,2],[3,7,2,3],[14,7,2,3],[28,7,2,4],[35,9,2,2],[9,13,3,2],[22,13,3,2],[4,17,2,2],[16,17,2,2],[30,16,2,3],[11,22,3,2],[22,22,3,2]],start:[3,3],toy:[34,23]},
{name:'Le Bunker',rooms:{A:[1,1,5,5],B:[10,1,6,6],C:[20,1,6,6],D:[30,1,8,6],E:[1,10,6,6],F:[11,11,8,6],G:[23,10,6,7],H:[32,11,6,6],I:[3,20,8,6],J:[26,20,11,6]},corr:[[6,3,4,1],[16,3,4,1],[26,4,4,1],[3,6,1,4],[13,7,1,4],[24,7,1,3],[34,7,1,4],[7,13,4,1],[19,13,4,1],[29,13,3,1],[3,16,1,4],[11,23,15,1],[34,17,1,3]],start:[3,3],toy:[34,23]}];
function build(i){MAP=MAPS[i];ROOMS=MAP.rooms;CORR=MAP.corr;g=Array.from({length:R},()=>Array(C).fill(1));
[...Object.values(ROOMS),...CORR].forEach(([x,y,w,h])=>{for(let j=y;j<y+h;j++)for(let k=x;k<x+w;k++)g[j][k]=0});seed=7;
for(const n in ROOMS){if(n=='A'||n=='J')continue;const[x,y,w,h]=ROOMS[n];for(let k=0;k<3;k++)g[y+2+Math.floor(rs()*(h-4))][x+2+Math.floor(rs()*(w-4))]=2}}
let seed=7;const rs=()=>(seed=seed*16807%2147483647)/2147483647;
build(0);
const fr=(x,y)=>x>=0&&y>=0&&x<C&&y<R&&g[y][x]==0;
const hit=(x,y,r=11)=>[[-r,-r],[r,-r],[-r,r],[r,r]].some(([a,b])=>!fr(Math.floor((x+a)/T),Math.floor((y+b)/T)));
function rt(room){const[x,y,w,h]=ROOMS[room];for(;;){const tx=x+Math.floor(Math.random()*w),ty=y+Math.floor(Math.random()*h);if(fr(tx,ty))return[tx,ty]}}
function rtS(room){const[x,y,w,h]=ROOMS[room];for(;;){const tx=x+Math.floor(rs()*w),ty=y+Math.floor(rs()*h);if(fr(tx,ty))return[tx,ty]}}
function los(x1,y1,x2,y2){const n=Math.ceil(hyp(x2-x1,y2-y1)/12);for(let i=1;i<n;i++)if(!fr(Math.floor((x1+(x2-x1)*i/n)/T),Math.floor((y1+(y2-y1)*i/n)/T)))return false;return true}
function bfs(a,b){const q=[a],pv=new Map([[a[0]+a[1]*C,null]]);
for(let i=0;i<q.length;i++){const c=q[i];if(c[0]==b[0]&&c[1]==b[1])break;
for(const[dx,dy]of[[1,0],[-1,0],[0,1],[0,-1]]){const n=[c[0]+dx,c[1]+dy],k=n[0]+n[1]*C;if(!pv.has(k)&&fr(n[0],n[1])){pv.set(k,c);q.push(n)}}}
if(!pv.has(b[0]+b[1]*C))return null;let c=b;for(;;){const p=pv.get(c[0]+c[1]*C);if(p[0]==a[0]&&p[1]==a[1])return c;c=p}}
/* ---------- DESSIN ---------- */
const ci=(x,y,r,c)=>{X.fillStyle=c;X.beginPath();X.arc(x,y,Math.max(.5,r),0,7);X.fill()};
const rc=(x,y,w,h,c)=>{X.fillStyle=c;X.fillRect(x,y,w,h)};
const el=(x,y,w,h,c)=>{X.fillStyle=c;X.beginPath();X.ellipse(x,y,Math.max(.5,w),Math.max(.5,h),0,0,7);X.fill()};
const tx=(t,x,y,s,c,al='center')=>{X.font=`bold ${s}px Arial`;X.fillStyle=c;X.textAlign=al;X.fillText(t,x,y)};
function heart(cx,cy,sz,c){X.fillStyle=c;X.beginPath();for(let i=0;i<=40;i++){const t=i/40*6.2832,x=16*Math.sin(t)**3,y=-(13*Math.cos(t)-5*Math.cos(2*t)-2*Math.cos(3*t)-Math.cos(4*t));i?X.lineTo(cx+x*sz/16,cy+y*sz/16):X.moveTo(cx+x*sz/16,cy+y*sz/16)}X.fill()}
function face(cx,cy,r,col,kind){
if(kind=='collector'){rc(cx-r*.55,cy-r*1.45,r*1.1,r*.55,'#0f0f0f');rc(cx-r*.85,cy-r*.95,r*1.7,r*.12,'#0f0f0f')}
el(cx,cy,r*.75,r,col);
for(const s of[-1,1]){el(cx+s*r*.35,cy-r*.1,r*.2,r*.25,'#000');ci(cx+s*r*.35,cy-r*.1,Math.max(3,r*.07),'#f00')}
X.fillStyle='#000';X.beginPath();X.moveTo(cx-r*.45,cy+r*.3);X.lineTo(cx+r*.45,cy+r*.3);X.lineTo(cx+r*.3,cy+r*.85);X.lineTo(cx-r*.3,cy+r*.85);X.fill();
X.fillStyle='#f0f0e6';for(let i=0;i<8;i++){const x=cx-r*.4+i*r*.1;X.beginPath();X.moveTo(x,cy+r*.3);X.lineTo(x+r*.1,cy+r*.3);X.lineTo(x+r*.05,cy+r*.52);X.fill()}}
/* silhouettes 3D (h = hauteur à l’écran, b = pied) */
const SC=document.createElement('canvas'),SX=SC.getContext('2d');
function heat(x,b,h,a){X.save();X.globalAlpha=a;X.translate(x,b-h*.5);X.scale(.42,1);const g=X.createRadialGradient(0,0,0,0,0,h*.55);g.addColorStop(0,'#fffbe0');g.addColorStop(.35,'#ffc21a');g.addColorStop(.7,'#ff4a00');g.addColorStop(1,'rgba(160,0,60,0)');X.fillStyle=g;X.beginPath();X.arc(0,0,h*.55,0,7);X.fill();X.restore();X.globalAlpha=a;ci(x,b-h*.88,h*.07,'#fff8c8');X.globalAlpha=1}
function figShaded(kind,xs,b,h,al){const w=Math.ceil(h*1.2),hh=Math.ceil(h*1.15);if(w<6||w>900||hh>900){X.globalAlpha=al;figure(kind,xs,b,h,false);X.globalAlpha=1;return}
SC.width=w;SC.height=hh;const oX=X;X=SX;figure(kind,w/2,hh-4,h,false);X=oX;
SX.globalCompositeOperation='source-atop';let g=SX.createLinearGradient(0,0,0,hh);g.addColorStop(0,'rgba(255,230,190,.12)');g.addColorStop(.5,'rgba(0,0,0,0)');g.addColorStop(1,'rgba(0,0,0,.5)');SX.fillStyle=g;SX.fillRect(0,0,w,hh);
g=SX.createLinearGradient(0,0,w,0);g.addColorStop(0,'rgba(0,0,0,.35)');g.addColorStop(.4,'rgba(0,0,0,0)');g.addColorStop(.75,'rgba(255,240,210,.12)');g.addColorStop(1,'rgba(0,0,0,.3)');SX.fillStyle=g;SX.fillRect(0,0,w,hh);SX.globalCompositeOperation='source-over';
X.globalAlpha=al;X.drawImage(SC,xs-w/2,b-hh+4);X.globalAlpha=1}
function figure(kind,x,b,h,eyes){const u=h/100,t=performance.now()/1000;
if(kind=='lytrho'){lytrho(x,b,u,t);return}if(kind=='old1'){el(x,b-45*u,13*u,45*u,'#19191e');X.strokeStyle='#26262d';X.lineWidth=Math.max(1,4*u);for(const s of[-1,1]){X.beginPath();X.moveTo(x+s*12*u,b-70*u);X.lineTo(x+s*(30+4*Math.sin(t*5))*u,b-8*u);X.stroke()}
el(x,b-90*u,9*u,12*u,'#c8c8be');el(x-4*u,b-92*u,2.5*u,3.5*u,'#000');el(x+4*u,b-92*u,2.5*u,3.5*u,'#000');rc(x-5*u,b-82*u,10*u,2*u,'#000');
if(eyes){ci(x-4*u,b-92*u,2*u,'#fff');ci(x+4*u,b-92*u,2*u,'#fff')}}
else if(kind=='collector'){collector(x,b,u,t);return}else if(kind=='old2'){rc(x-10*u,b-30*u,20*u,30*u,'#2a2a40');rc(x-14*u,b-68*u,28*u,40*u,'#3c3c5a');rc(x-2*u,b-66*u,4*u,20*u,'#aa0000');ci(x,b-78*u,11*u,'#d7aa8c');rc(x-9*u,b-102*u,18*u,16*u,'#0a0a0a');rc(x-15*u,b-88*u,30*u,4*u,'#0a0a0a');el(x,b-72*u,5*u,3*u,'#780000');ci(x+24*u,b-35*u,10*u,'#bea028');tx('$',x+24*u,b-30*u,Math.max(6,14*u),'#000');
ci(x-4*u,b-79*u,2*u,eyes?'#f00':'#a00');ci(x+4*u,b-79*u,2*u,eyes?'#f00':'#a00')}
else if(kind=='toy'){const s=h;ci(x,b-s*.5,s*.5,'#b07a3c');ci(x-s*.4,b-s*.95,s*.22,'#b07a3c');ci(x+s*.4,b-s*.95,s*.22,'#b07a3c');ci(x,b-s*1.4,s*.38,'#b07a3c');ci(x-s*.12,b-s*1.45,s*.05,'#000');ci(x+s*.12,b-s*1.45,s*.05,'#000')}
else if(kind=='lamp'){const f=.8+.2*Math.sin(t*9+x*.1),by=b-h*.78;X.strokeStyle='#222';X.lineWidth=Math.max(1,h*.006);X.beginPath();X.moveTo(x,b-h);X.lineTo(x,by-h*.06);X.stroke();pg([[-9,0],[9,0],[4,-8],[-4,-8]],x,by-h*.06,h*.01,'#1a1a1a');const g=X.createRadialGradient(x,by,0,x,by,h*.3);g.addColorStop(0,`rgba(255,235,170,${.9*f})`);g.addColorStop(.3,`rgba(255,190,90,${.35*f})`);g.addColorStop(1,'rgba(255,150,40,0)');X.fillStyle=g;X.beginPath();X.arc(x,by,h*.3,0,7);X.fill();ci(x,by,h*.02,'#fff4c8')}
else if(kind=='chair'){rc(x-h*.28,b-h*.45,h*.56,h*.07,'#5a3b22');rc(x-h*.26,b-h*.45,h*.05,h*.45,'#4a2f1a');rc(x+h*.21,b-h*.45,h*.05,h*.45,'#4a2f1a');rc(x-h*.26,b-h,h*.05,h*.55,'#4a2f1a');rc(x+h*.21,b-h*.85,h*.05,h*.4,'#4a2f1a');for(let i=0;i<3;i++)rc(x-h*.22,b-h*(.95-i*.14),h*.44,h*.04,'#5a3b22')}
else if(kind=='barrel'){el(x,b-h*.5,h*.3,h*.5,'#5b4a3a');X.strokeStyle='#2a2018';X.lineWidth=Math.max(1,h*.03);for(const y of[.2,.5,.8]){X.beginPath();X.ellipse(x,b-h*y,h*.3,h*.05,0,0,7);X.stroke()}el(x,b-h,h*.3,h*.06,'#3a2e22');X.fillStyle='rgba(140,60,20,.35)';X.fillRect(x-h*.25,b-h*.8,h*.1,h*.6)}
else if(kind=='player'){rc(x-10*u,b-65*u,20*u,40*u,'#2a5a8a');rc(x-9*u,b-28*u,7*u,28*u,'#1f2f45');rc(x+2*u,b-28*u,7*u,28*u,'#1f2f45');ci(x,b-76*u,11*u,'#d7aa8c');ci(x+16*u,b-45*u,5*u,'#ffe98a')}
else if(kind=='table'){rc(x-h*.95,b-h,h*1.9,h*.12,'#7a5530');rc(x-h*.85,b-h*.88,h*.1,h*.88,'#5a3d20');rc(x+h*.75,b-h*.88,h*.1,h*.88,'#5a3d20');rc(x-h*.75,b-h*.88,h*1.5,h*.08,'#1a1008')}
else if(kind=='box'){rc(x-h*.6,b-h,h*1.2,h,'#a67c4a');rc(x-h*.6,b-h,h*1.2,h*.15,'#8c6538');rc(x-h*.05,b-h,h*.1,h,'#d8c9a8')}
else if(kind=='bat'){rc(x-h*.4,b-h,h*.8,h,'#d4b01c');rc(x-h*.25,b-h*1.15,h*.5,h*.18,'#888')}
else{rc(x-h*.03,b-h,h*.06,h,'#3cff5a');tx('SORTIE',x,b-h-8,Math.max(10,h*.25),'#3cff5a')}}
function drawScare(sc){const e=sc.dur-sc.t,k=sc.kind,sh=()=>rnd(-14,14),r=150+250*Math.min(1,e/.2),fl=Math.floor(e*30)%2;
if(k=='eyes'){rc(0,0,W,H,'#000');for(let i=0;i<16;i++){const x=(i*173+e*60*(i%3+1))%W,y=(i*97)%H,s=6+(i*7)%12;ci(x-s,y,s*.45,'#f00');ci(x+s,y,s*.45,'#f00')}tx('ILS T’OBSERVENT',W/2+sh(),H/2,44,'#a00');return}
if(k=='static'){rc(0,0,W,H,'#111');for(let i=0;i<500;i++){const v=rnd(60,255)|0;rc(rnd(0,W),rnd(0,H),rnd(10,90),rnd(2,6),`rgb(${v},${v},${v})`)}X.globalAlpha=.6;face(W/2+sh(),H/2,300,'#ddd','ghost');X.globalAlpha=1;return}
if(k=='hand'){rc(0,0,W,H,fl?'#000':'#400');X.strokeStyle='#d9d0c4';X.lineCap='round';X.lineWidth=46;const L=Math.min(W*.46,e*2600);for(let i=0;i<5;i++){const y=H*.25+i*H*.12;for(const s of[0,1]){X.beginPath();X.moveTo(s?W:0,y+sh());X.lineTo(s?W-L:L,H/2+(i-2)*34);X.stroke()}}return}
rc(0,0,W,H,fl?'#000':'#c80000');
if(k=='demon'){X.fillStyle='#200';for(const s of[-1,1]){X.beginPath();X.moveTo(W/2+s*r*.5,H/2-r*.6);X.lineTo(W/2+s*r,H/2-r*1.5);X.lineTo(W/2+s*r*.2,H/2-r*.9);X.fill()}}
X.save();X.translate(W/2+sh(),H/2+sh());if(k=='crawler')X.rotate(Math.PI);
face(0,0,r,{lytrho:'#cdcdbe',collector:'#d7aa8c',ghost:'#ebebeb',demon:'#a01010',crawler:'#ddd'}[k]||'#ebebeb',k);X.restore();
if(k=='collector')tx('DONNE TON ARGENT !!!',W/2+sh(),H-60,36,'#ff0')}
/* ---------- TEXTURES ---------- */
function mkTex(f){const c=document.createElement('canvas');c.width=c.height=64;f(c.getContext('2d'));return c}
const TEX={};
TEX.wall=mkTex(x=>{x.fillStyle='#3a332e';x.fillRect(0,0,64,64);for(let r=0;r<4;r++)for(let b=-1;b<2;b++){const bx=b*32+(r%2)*16,c=70+rnd(0,35)|0;x.fillStyle=`rgb(${c+30},${c-8},${c-24})`;x.fillRect(bx+1,r*16+1,30,14)}
for(let i=0;i<320;i++){x.fillStyle=`rgba(0,0,0,${rnd(.05,.28)})`;x.fillRect(rnd(0,64),rnd(0,64),rnd(1,3),rnd(1,3))}
x.strokeStyle='rgba(0,0,0,.35)';x.beginPath();x.moveTo(12,0);x.lineTo(16,20);x.lineTo(11,40);x.lineTo(18,64);x.stroke();
const g2=x.createLinearGradient(0,0,0,64);g2.addColorStop(0,'rgba(0,0,0,0)');g2.addColorStop(1,'rgba(25,0,0,.4)');x.fillStyle=g2;x.fillRect(0,0,64,64)});
TEX.crate=mkTex(x=>{x.fillStyle='#6b4527';x.fillRect(0,0,64,64);for(let i=0;i<8;i++){x.fillStyle=`rgba(0,0,0,${.1+.1*(i%2)})`;x.fillRect(0,i*8,64,1.5)}
x.strokeStyle='#3b2412';x.lineWidth=4;x.strokeRect(2,2,60,60);x.beginPath();x.moveTo(4,4);x.lineTo(60,60);x.moveTo(60,4);x.lineTo(4,60);x.stroke();
for(let i=0;i<200;i++){x.fillStyle=`rgba(0,0,0,${rnd(.05,.25)})`;x.fillRect(rnd(0,64),rnd(0,64),rnd(1,6),1)}});
TEX.floor=mkTex(x=>{x.fillStyle='#22170f';x.fillRect(0,0,64,64);for(let i=0;i<4;i++){const c=rnd(26,46)|0;x.fillStyle=`rgb(${c+14},${c},${c-10})`;x.fillRect(i*16+1,0,14,64)}
for(let i=0;i<280;i++){x.fillStyle=`rgba(0,0,0,${rnd(.05,.3)})`;x.fillRect(rnd(0,64),rnd(0,64),rnd(1,4),1)}});
TEX.ceil=mkTex(x=>{x.fillStyle='#1d1612';x.fillRect(0,0,64,64);x.fillStyle='#0f0a07';x.fillRect(0,26,64,10);for(let i=0;i<200;i++){x.fillStyle=`rgba(0,0,0,${rnd(.05,.3)})`;x.fillRect(rnd(0,64),rnd(0,64),rnd(1,3),rnd(1,3))}});
TEX.wall2=mkTex(x=>{x.fillStyle='#4a5a46';x.fillRect(0,0,64,64);for(let i=0;i<64;i+=8){x.fillStyle=i%16?'#3e4e3b':'#566650';x.fillRect(i,0,8,64)}
for(let i=0;i<5;i++){x.fillStyle='rgba(30,20,10,.55)';x.beginPath();x.moveTo(rnd(0,60),0);x.lineTo(rnd(0,60)+8,rnd(10,40);x.lineTo(rnd(0,60),0);x.fill()}
x.fillStyle='rgba(90,0,0,.5)';x.beginPath();x.ellipse(40,36,11,16,0,0,7);x.fill();for(let i=0;i<4;i++)x.fillRect(32+i*5,46,3,rnd(10,18));
for(let i=0;i<300;i++){x.fillStyle=`rgba(0,0,0,${rnd(.04,.2)})`;x.fillRect(rnd(0,64),rnd(0,64),rnd(1,3),rnd(1,3))}});
TEX.rags=mkTex(x=>{x.fillStyle='#5b5750';x.fillRect(0,0,64,64);for(let i=0;i<40;i++){const c=rnd(60,105)|0;x.fillStyle=`rgb(${c},${c-4},${c-10})`;x.fillRect(rnd(0,60),rnd(0,60),rnd(6,16),rnd(4,10))}
for(let i=0;i<64;i+=3){x.fillStyle='rgba(0,0,0,.12)';x.fillRect(i,0,1,64);x.fillRect(0,i,64,1)}
for(let i=0;i<6;i++){x.fillStyle=`rgba(${rnd(60,110)|0},0,0,.45)`;x.beginPath();x.arc(rnd(0,64),rnd(0,64),rnd(3,9),0,0,7);x.fill()}});
TEX.suit=mkTex(x=>{x.fillStyle='#1b2135';x.fillRect(0,0,64,64);for(let i=2;i<64;i+=8){x.fillStyle='#4a5a85';x.fillRect(i,0,1,64)}for(let i=0;i<200;i++){x.fillStyle=`rgba(0,0,0,${rnd(.05,.2)})`;x.fillRect(rnd(0,64),rnd(0,64),1,rnd(1,3))}});
TEX.skin=mkTex(x=>{x.fillStyle='#a79c91';x.fillRect(0,0,64,64);for(let i=0;i<30;i++){x.fillStyle=`rgba(${rnd(60,110)|0},${rnd(40,90)|0},${rnd(70,120)|0},.25)`;x.beginPath();x.arc(rnd(0,64),rnd(0,64),rnd(3,10),0,0,7);x.fill()}x.strokeStyle='rgba(40,50,110,.5)';for(let i=0;i<5;i++){x.beginPath();x.moveTo(rnd(0,64),0);x.bezierCurveTo(rnd(0,64),20,rnd(0,64),40,rnd(0,64),64);x.stroke()}});
const PT={};function pf(n,s,ox,oy){const p=PT[n]||(PT[n]=X.createPattern(TEX[n],'repeat'));p.setTransform(new DOMMatrix([s,0,0,s,ox,oy]));return p}
function pg(pts,x,b,u,f){X.fillStyle=f;X.beginPath();pts.forEach(([a,c],i)=>i?X.lineTo(x+a*u,b+c*u):X.moveTo(x+a*u,b+c*u));X.closePath();X.fill()}
function lytrho(x,b,u,t){const sw=Math.sin(t*3)*3,ru=Math.max(.2,u*.35),R=pf('rags',ru,x,b),Sk=pf('skin',ru,x,b);
rc(x-9*u,b-30*u,6*u,30*u,'#15151a');rc(x+3*u,b-30*u,6*u,30*u,'#15151a');rc(x-11*u,b-3*u,9*u,3*u,'#0a0a0a');rc(x+2*u,b-3*u,9*u,3*u,'#0a0a0a');
X.lineCap='round';for(const s of[-1,1]){X.strokeStyle=Sk;X.lineWidth=Math.max(1,5*u);X.beginPath();X.moveTo(x+s*14*u,b-68*u);X.lineTo(x+s*(24+sw*s)*u,b-40*u);X.lineTo(x+s*(30+sw)*u,b-8*u);X.stroke();X.strokeStyle='#cfc8bd';X.lineWidth=Math.max(1,1.4*u);for(let f=-1;f<=1;f++){X.beginPath();X.moveTo(x+s*(30+sw)*u,b-8*u);X.lineTo(x+s*(32+sw+f*3)*u,b-1*u);X.stroke()}}
pg([[-15,-72],[15,-72],[19,-20],[13,-26],[8,-18],[2,-27],[-4,-17],[-10,-26],[-14,-19],[-19,-22]],x,b,u,R);X.fillStyle='rgba(0,0,0,.35)';X.fillRect(x-2*u,b-72*u,4*u,40*u);
X.fillStyle=Sk;X.beginPath();X.ellipse(x,b-92*u,9*u,12*u,0,0,7);X.fill();X.fillStyle=R;X.fillRect(x-9*u,b-96*u,18*u,5*u);
el(x-4*u,b-92*u,2.6*u,3.6*u,'#000');el(x+4*u,b-92*u,2.6*u,3.6*u,'#000');
X.strokeStyle='#111';X.lineWidth=Math.max(1,.8*u);X.beginPath();X.moveTo(x-5*u,b-83*u);X.lineTo(x+5*u,b-83*u);for(let i=-4;i<=4;i+=2){X.moveTo(x+i*u,b-85*u);X.lineTo(x+i*u,b-81*u)}X.stroke();
X.strokeStyle='#0a0a0a';for(let i=0;i<9;i++){X.beginPath();X.moveTo(x+(i-4)*2*u,b-102*u);X.lineTo(x+(i-4)*2.6*u,b-84*u-(i%3)*3*u);X.stroke()}}
function collector(x,b,u,t){const S_=pf('suit',u*.35,x,b),Sk=pf('skin',u*.35,x,b),sw=Math.sin(t*6)*2;
pg([[-10,-30],[-1,-30],[-1,0],[-10,0]],x,b,u,S_);pg([[1,-30],[10,-30],[10,0],[1,0]],x,b,u,S_);rc(x-12*u,b-3*u,11*u,3*u,'#050505');rc(x+1*u,b-3*u,11*u,3*u,'#050505');
pg([[-15,-70],[15,-70],[17,-28],[-17,-28]],x,b,u,S_);pg([[-4,-70],[4,-70],[0,-48]],x,b,u,'#e8e4da');pg([[-1.5,-68],[1.5,-68],[2.5,-46],[0,-42],[-2.5,-46]],x,b,u,'#a00000');
X.strokeStyle='#000';X.lineWidth=Math.max(1,.8*u);X.beginPath();X.moveTo(x-4*u,b-70*u);X.lineTo(x-9*u,b-44*u);X.moveTo(x+4*u,b-70*u);X.lineTo(x+9*u,b-44*u);X.stroke();
X.lineCap='round';for(const s of[-1,1]){X.strokeStyle=S_;X.lineWidth=Math.max(1,6*u);X.beginPath();X.moveTo(x+s*15*u,b-66*u);X.lineTo(x+s*(21+sw*s*.5)*u,b-38*u);X.stroke();ci(x+s*(21+sw*s*.5)*u,b-35*u,3.2*u,'#d9cdbf')}
rc(x+17*u,b-34*u,14*u,10*u,'#3a2412');rc(x+21*u,b-37*u,6*u,3*u,'#2a1808');ci(x-22*u,b-31*u,8*u,'#bea028');tx('$',x-22*u,b-27*u,Math.max(6,11*u),'#000');
X.fillStyle=Sk;X.beginPath();X.ellipse(x,b-78*u,11*u,12.5*u,0,0,7);X.fill();ci(x-4*u,b-79*u,2.4*u,'#111');ci(x+4*u,b-79*u,2.4*u,'#111');
X.strokeStyle='#d4b020';X.lineWidth=Math.max(1,.8*u);X.beginPath();X.arc(x+4*u,b-79*u,4*u,0,7);X.stroke();
el(x,b-72*u,6*u,3.2*u,'#400');X.fillStyle='#e8e0c0';for(let i=-2;i<=2;i++)X.fillRect(x+i*2*u-.8*u,b-74.5*u,1.6*u,2*u);
rc(x-9*u,b-102*u,18*u,17*u,'#0a0a0a');rc(x-9*u,b-90*u,18*u,3*u,'#8a0f0f');rc(x-15*u,b-86*u,30*u,3.5*u,'#0a0a0a')}
function scareFX(e){X.globalCompositeOperation='lighter';X.globalAlpha=.28;X.drawImage(cv,-10,0);X.drawImage(cv,10,0);X.globalAlpha=1;X.globalCompositeOperation='source-over';
for(let i=0;i<260;i++){const v=rnd(0,255)|0;X.fillStyle=`rgba(${v},${v},${v},.18)`;X.fillRect(rnd(0,W),rnd(0,H),rnd(1,4),rnd(1,3))}
X.fillStyle='rgba(0,0,0,.18)';for(let y=0;y<H;y+=3)X.fillRect(0,y,W,1);
const g=X.createRadialGradient(W/2,H/2,H*.25,W/2,H/2,H*.8);g.addColorStop(0,'rgba(0,0,0,0)');g.addColorStop(1,'rgba(120,0,0,.7)');X.fillStyle=g;X.fillRect(0,0,W,H);
if(Math.random()<.4){X.fillStyle='rgba(255,255,255,.25)';X.fillRect(0,rnd(0,H),W,rnd(2,18))}}
function face(cx,cy,r,col,kind){const e=face.t||0,jaw=(.5+.5*Math.sin(e*26))*r*.16;
if(kind!='ghost'){X.strokeStyle='#08080a';X.lineWidth=Math.max(2,r*.022);for(let i=0;i<70;i++){const a=-Math.PI+i/70*Math.PI,x0=cx+Math.cos(a)*r*.72,y0=cy+Math.sin(a)*r*.95;X.beginPath();X.moveTo(x0,y0);X.quadraticCurveTo(x0+Math.cos(a)*r*.3,y0+Math.sin(a)*r*.15,x0+Math.cos(a)*r*.5+Math.sin(i)*r*.05,y0+r*(.25+.2*Math.abs(Math.sin(i*3))));X.stroke()}}
if(kind=='collector'){rc(cx-r*.55,cy-r*1.5,r*1.1,r*.6,'#0c0c0c');rc(cx-r*.55,cy-r*1.05,r*1.1,r*.12,'#8a0f0f');rc(cx-r*.9,cy-r*.92,r*1.8,r*.1,'#0c0c0c')}
if(kind=='demon'){X.fillStyle='#1a0000';for(const s of[-1,1]){X.beginPath();X.moveTo(cx+s*r*.45,cy-r*.8);X.quadraticCurveTo(cx+s*r*1.1,cy-r,cx+s*r*.95,cy-r*1.7);X.quadraticCurveTo(cx+s*r*.6,cy-r*1.1,cx+s*r*.2,cy-r*.9);X.fill()}}
const g=X.createRadialGradient(cx,cy-r*.2,r*.1,cx,cy,r);g.addColorStop(0,col);g.addColorStop(1,'#1a1210');X.fillStyle=g;X.beginPath();X.ellipse(cx,cy,r*.75,r,0,0,7);X.fill();
X.fillStyle='rgba(0,0,0,.38)';for(const s of[-1,1]){X.beginPath();X.ellipse(cx+s*r*.42,cy+r*.28,r*.16,r*.3,s*.2,0,0,7);X.fill()}
X.strokeStyle='rgba(120,20,30,.55)';X.lineWidth=Math.max(1,r*.008);for(let i=0;i<14;i++){const a=i*.9;X.beginPath();X.moveTo(cx+Math.cos(a)*r*.3,cy-r*.55+Math.sin(a)*r*.15);X.lineTo(cx+Math.cos(a)*r*.62,cy-r*.8+Math.sin(a*2)*r*.2);X.stroke()}
for(const s of[-1,1]){const ex=cx+s*r*.34,ey=cy-r*.1;el(ex,ey,r*.22,r*.27,'#050303');X.strokeStyle='rgba(200,0,0,.7)';for(let i=0;i<10;i++){const a=i/10*6.28;X.beginPath();X.moveTo(ex+Math.cos(a)*r*.1,ey+Math.sin(a)*r*.1);X.lineTo(ex+Math.cos(a)*r*.2,ey+Math.sin(a)*r*.25);X.stroke()}
const 
