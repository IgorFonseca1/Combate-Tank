import processing.sound.*;

PFont gameFont;
float playerX=120, playerY=700, angle=0;
float enemyX=1100, enemyY=200, enemyAngle=0;
boolean up, down, left, right, gameStarted, gameOver;
int score=0, lives=3;
int enemyStuckTimer=0;
float enemySteerOffset=0;

ArrayList<PVector> bullets=new ArrayList<PVector>();
ArrayList<Float>   bulletAngles=new ArrayList<Float>();
ArrayList<PVector> enemyBullets=new ArrayList<PVector>();
ArrayList<Float>   enemyBulletAngles=new ArrayList<Float>();
ArrayList<Particle> particles=new ArrayList<Particle>();
ArrayList<Particle> smoke=new ArrayList<Particle>();
ArrayList<PVector> walls=new ArrayList<PVector>();
ArrayList<PVector> stars=new ArrayList<PVector>();

class Particle {
  float x,y,vx,vy,life,maxLife,sz;
  int col;
  boolean isSmoke;

  Particle(float x,float y,float vx,float vy,float life,int col,float sz,boolean isSmoke){
    this.x=x; this.y=y; this.vx=vx; this.vy=vy;
    this.life=life; this.maxLife=life; this.col=col; this.sz=sz; this.isSmoke=isSmoke;
  }

  void update(){
    x+=vx; y+=vy; vx*=0.92; vy*=0.92;
    if(isSmoke){ vy-=0.05; sz+=0.25; }
    life--;
  }

  void display(){
    float a=map(life,0,maxLife,0,255);
    noStroke();
    fill(red(col),green(col),blue(col), isSmoke ? a*0.45 : a);
    ellipse(x,y,sz,sz);
  }

  boolean isDead(){ return life<=0; }
}

void setup(){
  size(1300,850); smooth(8);
  gameFont=createFont("Arial Bold",64,true);
  textFont(gameFont);
  buildMap();
  for(int i=0;i<120;i++) stars.add(new PVector(random(width),random(height)));
}

void buildMap(){
  walls.clear();
  for(int i=0;i<4;i++) walls.add(new PVector(560+i*60,340));
  for(int j=0;j<3;j++) walls.add(new PVector(220,250+j*60));
  for(int j=0;j<3;j++) walls.add(new PVector(1020,250+j*60));
  walls.add(new PVector(480,580)); walls.add(new PVector(480,640));
  walls.add(new PVector(780,580)); walls.add(new PVector(780,640));
  walls.add(new PVector(560,160)); walls.add(new PVector(620,160)); walls.add(new PVector(680,160));
}

void draw(){
  if(!gameStarted){ drawMenu(); return; }
  if(gameOver){ drawGameOver(); return; }

  background(28,50,32);
  stroke(45,75,45,70);
  for(int i=0;i<width;i+=40) line(i,80,i,height);
  for(int j=80;j<height;j+=40) line(0,j,width,j);
  noStroke();

  fill(15,20,30,240); rect(0,0,width,80);
  stroke(90,170,255,80); line(0,79,width,79); noStroke();
  for(int i=0;i<lives;i++) drawHeart(35+i*40,40,15);
  fill(255,230,90); textAlign(CENTER); textSize(30); text("PONTOS: "+score,width/2,50);
  fill(180,210,255); textSize(18); textAlign(RIGHT);
  text("W/S mover  •  A/D girar  •  ESPAÇO atirar",width-25,50);

  for(PVector w:walls) drawWall(w);

  movePlayer();
  drawTank(playerX,playerY,angle,color(70,255,130));
  if(up||down) spawnSmoke(playerX+30,playerY+30,angle+PI,color(180));

  moveEnemy();
  drawTank(enemyX,enemyY,enemyAngle,color(100,170,255));
  spawnSmoke(enemyX+30,enemyY+30,enemyAngle+PI,color(180));

  updatePlayerBullets();
  updateEnemyBullets();
  updateParticles();

  noFill();
  for(int i=0;i<40;i++){ stroke(0,0,0,4); rect(i,i,width-i*2,height-i*2); }
  noStroke();
}

void drawMenu(){
  background(8,12,22);
  for(PVector s:stars){ fill(255,random(70,180)); ellipse(s.x,s.y,2,2); }
  fill(20,40,80);  rect(width/2-340,120,680,140,25);
  fill(60,140,255); rect(width/2-332,128,664,124,22);
  fill(255); textAlign(CENTER); textSize(78); text("COMBAT TANK",width/2,210);
  fill(180,220,255); textSize(24); text("DESTRUA O TANQUE INIMIGO",width/2,320);
  fill(255,220,70); rect(width/2-190,450,380,75,25);
  fill(255,240,120); rect(width/2-184,456,368,63,20);
  fill(20); textSize(32); text("ENTER PARA JOGAR",width/2,500);
  fill(180); textSize(18); text("W/S mover  •  A/D girar  •  ESPAÇO atirar",width/2,620);
}

void drawGameOver(){
  background(10,5,10);
  for(int i=0;i<15;i++){ fill(255,40,40,20); ellipse(width/2,height/2,i*80,i*80); }
  fill(255,60,60); textAlign(CENTER); textSize(96); text("GAME OVER",width/2,280);
  fill(255); textSize(38); text("PONTOS: "+score,width/2,370);
  fill(255,220,80); textSize(28); text("PRESSIONE S PARA RECOMEÇAR",width/2,470);
}

void drawWall(PVector w){
  fill(0,60);      rect(w.x+6,w.y+6,60,60,8);
  fill(95,50,25);  rect(w.x,w.y,60,60,8);
  fill(175,90,45); rect(w.x+3,w.y+3,54,54,6);
  stroke(110,55,25,160);
  line(w.x+3,w.y+30,w.x+57,w.y+30); line(w.x+30,w.y+3,w.x+30,w.y+30);
  line(w.x+15,w.y+30,w.x+15,w.y+57); line(w.x+45,w.y+30,w.x+45,w.y+57);
  noStroke(); fill(255,180,120,70); rect(w.x+3,w.y+3,54,12,3);
}

void drawHeart(float x,float y,float r){
  noStroke(); fill(255,50,80);
  ellipse(x-r/2,y-r/3,r,r); ellipse(x+r/2,y-r/3,r,r);
  triangle(x-r,y-r/3+2, x+r,y-r/3+2, x,y+r);
  fill(255,180); ellipse(x-3,y-5,4,4);
}

void drawTank(float x,float y,float ang,int c){
  pushMatrix();
  translate(x+30,y+30); rotate(ang);
  fill(0,80); ellipse(5,7,60,44);
  fill(red(c)*0.35,green(c)*0.35,blue(c)*0.35);
  rect(-30,-22,60,14,4); rect(-30,8,60,14,4);
  stroke(0,70);
  for(int i=-25;i<30;i+=8){ line(i,-22,i,-8); line(i,8,i,22); }
  noStroke();
  fill(c); rect(-24,-18,48,36,10);
  fill(255,40); rect(-18,-14,24,10,5);
  fill(red(c)*0.75,green(c)*0.75,blue(c)*0.75); ellipse(0,0,30,30);
  fill(50); ellipse(0,0,12,12);
  fill(210); rect(0,-5,46,10,5);
  fill(120); rect(38,-6,12,12,3);
  popMatrix();
}

void movePlayer(){
  if(left&&!right) angle-=0.05;
  if(right&&!left) angle+=0.05;
  float nx=playerX, ny=playerY;
  if(up){   nx+=cos(angle)*3; ny+=sin(angle)*3; }
  if(down){  nx-=cos(angle)*3; ny-=sin(angle)*3; }
  if(!colideParede(nx,ny)){ playerX=nx; playerY=ny; }
  playerX=constrain(playerX,0,width-60);
  playerY=constrain(playerY,80,height-60);
}

void moveEnemy(){
  float dx=playerX-enemyX, dy=playerY-enemyY;
  float desired=atan2(dy,dx)+enemySteerOffset;
  float diff=atan2(sin(desired-enemyAngle),cos(desired-enemyAngle));
  enemyAngle+=diff*0.06;
  float tx=enemyX+cos(enemyAngle)*2.2, ty=enemyY+sin(enemyAngle)*2.2;
  if(!colideParede(tx,ty)){
    enemyX=tx; enemyY=ty;
    if(enemyStuckTimer>0&&--enemyStuckTimer==0) enemySteerOffset=0;
  } else {
    enemyStuckTimer+=3;
    if(enemyStuckTimer>120){
      enemySteerOffset=random(-PI*0.7,PI*0.7);
      enemyStuckTimer=40;
      enemyX-=cos(enemyAngle)*4; enemyY-=sin(enemyAngle)*4;
    } else if(enemyStuckTimer%30==0){
      enemySteerOffset=(enemySteerOffset==0)?HALF_PI*(random(1)>0.5?1:-1):-enemySteerOffset;
    }
  }
  enemyX=constrain(enemyX,0,width-60);
  enemyY=constrain(enemyY,80,height-60);
}

void updatePlayerBullets(){
  for(int i=bullets.size()-1;i>=0;i--){
    PVector b=bullets.get(i); float a=bulletAngles.get(i);
    b.x+=cos(a)*9; b.y+=sin(a)*9;
    spawnBulletTrail(b.x,b.y,color(255,220,80));
    if(colidiuParede(b.x,b.y,6)){
      explode(b.x,b.y,color(255,180,80),false); tocarExplosaoPequena();
      bullets.remove(i); bulletAngles.remove(i); continue;
    }
    fill(255,220,80,60); ellipse(b.x,b.y,20,20);
    fill(255,240,130);   ellipse(b.x,b.y,10,10);
    if(dist(b.x,b.y,enemyX+30,enemyY+30)<32){
      score+=10; explode(enemyX+30,enemyY+30,color(255,120,20),true); tocarExplosaoGrande();
      enemyX=random(800,1200); enemyY=random(150,700);
      bullets.remove(i); bulletAngles.remove(i);
    }
  }
}

void updateEnemyBullets(){
  if(frameCount%90==0){
    float tx=enemyX+30+cos(enemyAngle)*42, ty=enemyY+30+sin(enemyAngle)*42;
    enemyBullets.add(new PVector(tx,ty)); enemyBulletAngles.add(enemyAngle);
    tocarTiroInimigo();
  }
  for(int i=enemyBullets.size()-1;i>=0;i--){
    PVector b=enemyBullets.get(i); float a=enemyBulletAngles.get(i);
    b.x+=cos(a)*7; b.y+=sin(a)*7;
    spawnBulletTrail(b.x,b.y,color(255,90,90));
    if(colidiuParede(b.x,b.y,6)){
      explode(b.x,b.y,color(180,90,80),false); tocarExplosaoPequena();
      enemyBullets.remove(i); enemyBulletAngles.remove(i); continue;
    }
    fill(255,90,90,70); ellipse(b.x,b.y,20,20);
    fill(255,90,90);    ellipse(b.x,b.y,10,10);
    if(dist(b.x,b.y,playerX+30,playerY+30)<32){
      lives--; explode(playerX+30,playerY+30,color(255,120,20),true); tocarExplosaoGrande();
      if(lives<=0){ gameOver=true; tocarGameOver(); }
      playerX=120; playerY=700; angle=0;
      enemyBullets.remove(i); enemyBulletAngles.remove(i);
    }
  }
}

void spawnBulletTrail(float x,float y,int col){
  if(frameCount%2!=0) return;
  particles.add(new Particle(x,y,random(-0.4,0.4),random(-0.4,0.4),14,col,5,false));
}

void spawnSmoke(float x,float y,float ang,int col){
  if(frameCount%5!=0) return;
  float vx=cos(ang)*random(0.5,1.5)+random(-0.3,0.3);
  float vy=sin(ang)*random(0.5,1.5)+random(-0.3,0.3);
  smoke.add(new Particle(x,y,vx,vy,40,col,8,true));
}

void explode(float x,float y,int col,boolean big){
  int count=big?45:16; float maxSpd=big?6:3;
  for(int i=0;i<count;i++){
    float a=random(TWO_PI), spd=random(1,maxSpd);
    int pc=lerpColor(color(255,80,10),color(255,230,60),random(1));
    particles.add(new Particle(x,y,cos(a)*spd,sin(a)*spd,random(20,big?60:30),pc,random(3,big?15:7),false));
  }
}

void updateParticles(){
  for(int i=smoke.size()-1;i>=0;i--){
    Particle p=smoke.get(i); p.update(); p.display(); if(p.isDead()) smoke.remove(i);
  }
  for(int i=particles.size()-1;i>=0;i--){
    Particle p=particles.get(i); p.update(); p.display(); if(p.isDead()) particles.remove(i);
  }
}

boolean colidiuParede(float x,float y,float r){
  for(PVector w:walls)
    if(x+r>w.x && x-r<w.x+60 && y+r>w.y && y-r<w.y+60) return true;
  return false;
}

boolean colideParede(float x,float y){
  for(PVector w:walls)
    if(x+60>w.x && x<w.x+60 && y+60>w.y && y<w.y+60) return true;
  return false;
}

void keyPressed(){
  if(keyCode==ENTER&&!gameStarted) gameStarted=true;
  if(key=='w'||key=='W') up=true;
  if(key=='s'||key=='S') down=true;
  if(key=='a'||key=='A') left=true;
  if(key=='d'||key=='D') right=true;
  if(gameOver&&(key=='s'||key=='S')) restartGame();
  if(key==' '&&gameStarted&&!gameOver){
    float tx=playerX+30+cos(angle)*44, ty=playerY+30+sin(angle)*44;
    bullets.add(new PVector(tx,ty)); bulletAngles.add(angle);
    explode(tx,ty,color(255,220,120),false); tocarTiroJogador();
  }
}

void keyReleased(){
  if(key=='w'||key=='W') up=false;
  if(key=='s'||key=='S') down=false;
  if(key=='a'||key=='A') left=false;
  if(key=='d'||key=='D') right=false;
}

void restartGame(){
  score=0; lives=3; gameOver=false;
  playerX=120; playerY=700; angle=0;
  enemyX=1100; enemyY=200; enemyAngle=0;
  bullets.clear(); bulletAngles.clear();
  enemyBullets.clear(); enemyBulletAngles.clear();
  particles.clear(); smoke.clear();
  enemyStuckTimer=0; enemySteerOffset=0;
}

void tocarTiroJogador(){
  new Thread(()->{
    SinOsc s=new SinOsc(this); s.play(880,0.35); delay(40); s.freq(660); delay(30); s.stop();
  }).start();
}

void tocarTiroInimigo(){
  new Thread(()->{
    SinOsc s=new SinOsc(this); s.play(330,0.25); delay(60); s.stop();
  }).start();
}

void tocarExplosaoGrande(){
  new Thread(()->{
    WhiteNoise n=new WhiteNoise(this); SinOsc s=new SinOsc(this);
    n.play(0.5); s.play(200,0.4);
    for(int i=200;i>40;i-=8){ s.freq(i); delay(15); }
    n.stop(); s.stop();
  }).start();
}

void tocarExplosaoPequena(){
  new Thread(()->{
    WhiteNoise n=new WhiteNoise(this); n.play(0.3); delay(80); n.stop();
  }).start();
}

void tocarGameOver(){
  new Thread(()->{
    int[] notas={440,392,349,294,262}; SinOsc s=new SinOsc(this);
    for(int n:notas){ s.play(n,0.5); delay(200); }
    s.stop();
  }).start();
}
