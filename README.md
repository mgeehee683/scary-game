CreepyTown/
├─ index.html
├─ README.md
├─ main.js
├─ package.json (optional)
├─ assets/
│ ├─ player.png
│ ├─ enemy.png
│ ├─ tileset.png
│ ├─ sfx/footsteps.wav
│ ├─ sfx/scream.wav
│ ├─ music/ambience.mp3
│ └─ ...
└─ data/
└─ map.json (optional)
# using Python 3
python -m http.server 8000
# then visit http://localhost:8000
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>CreepyTown — Escape</title>
<style>
html,body { height:100%; margin:0; background:#000; }
#game { width:100%; height:100vh; display:block; }
.notice { position:fixed; left:8px; top:8px; color:#ccc; font-family:Inter, system-ui, sans-serif; z-index:999; }
</style>
</head>
<body>
<div class="notice">WASD/Arrows to move • Space to toggle flashlight • Shift to sprint</div>
<div id="game"></div>
<script src="https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.min.js"></script>
<script src="main.js"></script>
</body>
</html>
/* main.js — Phaser 3 game. Single-file for simplicity. */
const dy = this.player.y - e.y;
const dist = Math.sqrt(dx*dx+dy*dy);
if (dist < e.detectionRadius && !this.player.isHidden){
// chase
const vx = dx/dist * (e.speed + (dist<120?20:0));
const vy = dy/dist * (e.speed + (dist<120?20:0));
e.body.setVelocity(vx,vy);
e.anims.play('enemy_walk', true);
// scary audio when very close
if (dist<80 && !this._screamPlayed){ this.sound.play('scream', { volume:0.6 }); this._screamPlayed=true; }
} else {
// patrol slowly
e.body.setVelocity(Math.cos(e.patrolAngle)*e.speed*0.2, Math.sin(e.patrolAngle)*e.speed*0.2);
e.patrolAngle += 0.002;
e.anims.play('enemy_walk', true);
}
});


// draw lighting mask
this.drawLighting();
}


drawLighting(){
// faster: render a black rectangle then cut a circle around player to simulate flashlight
const cam = this.cameras.main;
const width = this.physics.world.bounds.width;
const height = this.physics.world.bounds.height;
this.lightTexture.clear();
// fill with full black
this.lightTexture.fill(0x000000, 0.95);


if (this.flashOn){
const px = this.player.x;
const py = this.player.y;
// gradient circle
const g = this.make.graphics({ x:0, y:0, add:false });
const r = this.flashRadius;
const steps = 24;
for(let i=steps;i>0;i--){
const alpha = 0.04 * (i);
g.fillStyle(0xffffff, alpha);
g.fillCircle(px, py, r * (i/steps));
}
this.lightTexture.draw(g, 0, 0);
g.destroy();
}


// reduce darkness where enemies are in the light (so they are visible)
this.enemies.getChildren().forEach(e=>{
const dx = e.x - this.player.x; const dy = e.y - this.player.y;
const dist = Math.sqrt(dx*dx+dy*dy);
if (dist < this.flashRadius){
const g = this.make.graphics({ add:false });
g.fillStyle(0xffffff, 0.12);
g.fillCircle(e.x, e.y, 50);
this.lightTexture.draw(g,0,0);
g.destroy();
}
});


// apply mask to darkness rectangle
this.darkness.setMask(new Phaser.Display.Masks.BitmapMask(this, this.lightTexture));
this.darkness.setDepth(50);
}


playerCaught(){
// simple caught behavior
this.cameras.main.flash(600,255,0,0);
this.amb.stop();
this.sound.play('scream');
this.scene.restart();
}


win(){
this.amb.stop();
this.add.text(this.cameras.main.midPoint.x - 180, this.cameras.main.midPoint.y, 'You escaped! — Refresh to play again', { font:'28px monospace', fill:'#fff' }).setScrollFactor(0).setDepth(200);
this.scene.pause();
}
}


const config = {
type: Phaser.AUTO,
parent: 'game',
width: 960,
height: 640,
physics: { default: 'arcade', arcade: { debug:false } },
scene: [ BootScene, PreloadScene, TownScene ]
};


const game = new Phaser.Game(config);


// OPTIONAL: expose for debugging
window.__game = game;
