# YAPAY ZEKAYI KULLANARAK PROJEME YAPTIĞIM EKLEMELER

## CHATGPT

### Sorgu 1:
-asepriteda yaptığım bir animasyonu js ve canvas kullanarak tasarladığım bir oyuna nasıl aktarırırm
```
Aseprite'de yaptığın bir animasyonu JavaScript ve HTML5 Canvas kullandığın bir oyuna aktarmak için aşağıdaki adımları takip edebilirsin:

🎨 1. Aseprite Animasyonunu Dışa Aktar
Aseprite'den animasyonunu dışa aktarmak için:

A. Sprite Sheet (Çerçeve Atlası) Olarak Kaydet
File > Export Sprite Sheet seçeneğini seç.

Ayarlarda:

Sheet type: Horizontal veya Packed (tercihine göre)

Include frame data: JSON Array (bu, her çerçevenin koordinat ve süresini verir)

Bir .png sprite sheet ve bir .json dosyası elde edeceksin.

B. Alternatif: Her kareyi ayrı ayrı .png olarak dışa aktarmak (tercih etmezsen .json kullanmazsın)
💻 2. JavaScript ile Sprite Sheet'i Kullanmak
Gerekli varlıkları yükle:
js
Kopyala
Düzenle
const spriteImage = new Image();
spriteImage.src = 'assets/character_spritesheet.png';

let animationData;
fetch('assets/character_spritesheet.json')
  .then(res => res.json())
  .then(data => {
    animationData = data.frames;
    // oyunu başlat
  });
Sprite sheet'e göre frame'leri çizmek:
js
Kopyala
Düzenle
let currentFrame = 0;
let frameElapsed = 0;
const frameSpeed = 100; // ms cinsinden (istenirse JSON'dan alınabilir)

function drawFrame(ctx, deltaTime) {
  frameElapsed += deltaTime;
  if (frameElapsed > frameSpeed) {
    currentFrame = (currentFrame + 1) % animationData.length;
    frameElapsed = 0;
  }

  const frame = animationData[currentFrame].frame;
  ctx.drawImage(
    spriteImage,
    frame.x, frame.y, frame.w, frame.h,
    characterX, characterY, frame.w, frame.h
  );
}
Eğer JSON formatın farklıysa (aseprite bazen farklı versiyonlar üretir), frame gibi alanlara erişmeden önce yapıyı konsolda incele.

🛠 Alternatif: JSON Kullanmak İstemiyorsan
Aseprite'te sprite sheet'i sabit boyutta dışa aktar ve her frame'in boyutunu bildiğini varsay:

js
Kopyala
Düzenle
const frameWidth = 64;
const frameHeight = 64;
const totalFrames = 6;

function drawSimpleSprite(ctx, deltaTime) {
  frameElapsed += deltaTime;
  if (frameElapsed > frameSpeed) {
    currentFrame = (currentFrame + 1) % totalFrames;
    frameElapsed = 0;
  }

  ctx.drawImage(
    spriteImage,
    currentFrame * frameWidth, 0,
    frameWidth, frameHeight,
    characterX, characterY,
    frameWidth, frameHeight
  );
}
✅ Ekstra: Aseprite JSON Parser'ı Yazmak İstersen
Aseprite JSON çıktısı genellikle şöyle görünür:

json
Kopyala
Düzenle
{
  "frames": {
    "run_0.png": {
      "frame": {"x":0, "y":0, "w":64, "h":64},
      "duration": 100
    },
    ...
  }
}
Bu durumda Object.values(data.frames) ile çerçeveleri sıraya koyabilirsin.
```
## GROK
### Sorgu 1
-benim karakterim düşmana vurmuyor sebebi nedir. Elimdeki Güncel kod bu:
```
<style>
  body {
    background: black;
  }
</style> 
<canvas></canvas> 
<script>
  const canvas = document.querySelector('canvas');
  const c = canvas.getContext('2d');

  canvas.width = 1920;
  canvas.height = 932;

  const gravity = 0.5;

  class Sprite {
    constructor({ position, imageSrc }) {
      this.position = position;
      this.image = new Image();
      this.image.src = imageSrc;
      this.loaded = false;
      this.image.onload = () => {
        this.loaded = true;
      };
    }

    draw() {
      if (!this.image || !this.loaded) return;
      c.drawImage(this.image, this.position.x, this.position.y, canvas.width, canvas.height);
    }

    update() {
      this.draw();
    }
  }

  class Character {
    constructor({ position, animations }) {
      this.position = position;
      this.velocity = { x: 0, y: 1 };
      this.width = 120;
      this.height = 120;
      this.animations = animations;
      this.currentAnimation = 'idle';
      this.frameIndex = 0;
      this.frameCount = 0;
      this.facingRight = true;
      this.health = 100; // Can sistemi
      this.isAlive = true;
    }

    draw() {
      if (!this.isAlive) return;
      const anim = this.animations[this.currentAnimation];
      if (!anim.image || !anim.loaded) return;

      c.save();
      if (!this.facingRight) {
        c.scale(-1, 1);
        c.translate(-canvas.width, 0);
      }
      c.drawImage(
        anim.image,
        this.frameIndex * anim.frameWidth,
        0,
        anim.frameWidth,
        anim.frameHeight,
        this.facingRight ? this.position.x : canvas.width - this.position.x - this.width,
        this.position.y,
        this.width,
        this.height
      );
      c.restore();
    }

    takeDamage(amount) {
      if (!this.isAlive) return;
      this.health -= amount;
      if (this.health <= 0) {
        this.health = 0;
        this.isAlive = false;
        this.currentAnimation = 'idle';
        this.velocity.x = 0;
      } else {
        this.setAnimation('hurt');
      }
    }

    update() {
      if (!this.isAlive) return;

      const anim = this.animations[this.currentAnimation];
      this.frameCount++;
      if (this.frameCount >= 60 / anim.frameRate) {
        this.frameIndex = (this.frameIndex + 1) % anim.frameCount;
        this.frameCount = 0;

        if ((this.currentAnimation === 'attack' || this.currentAnimation === 'jump' || this.currentAnimation === 'hurt') && this.frameIndex === 0) {
          this.currentAnimation = 'idle';
        }
      }

      this.position.x += this.velocity.x;
      this.position.y += this.velocity.y;

      if (this.position.y + this.height + this.velocity.y >= collisionBlock.position.y) {
        this.velocity.y = 0;
        this.position.y = collisionBlock.position.y - this.height;
        if (this.currentAnimation === 'jump') {
          this.currentAnimation = 'idle';
        }
      } else {
        this.velocity.y += gravity;
      }

      this.draw();
    }

    setAnimation(animation) {
      if (this.currentAnimation !== animation) {
        this.currentAnimation = animation;
        this.frameIndex = 0;
        this.frameCount = 0;
      }
    }
  }

  class Enemy {
    constructor({ position, animations }) {
      this.position = position;
      this.velocity = { x: 0, y: 0.3 };
      this.width = 300;
      this.height = 300;
      this.animations = animations;
      this.currentAnimation = 'idle';
      this.frameIndex = 0;
      this.frameCount = 0;
      this.health = 100; // Can sistemi
      this.isAlive = true;
      this.attackCooldown = 1000; // Saldırı gecikmesi
    }

    draw() {
      if (!this.isAlive) return;
      const anim = this.animations[this.currentAnimation];
      if (!anim.image || !anim.loaded) return;
      c.save();
      const facingRight = this.position.x > character.position.x;
      if (!facingRight) {
        c.scale(-1, 1);
        c.translate(-canvas.width, 0);
      }
      c.drawImage(
        anim.image,
        this.frameIndex * anim.frameWidth,
        0,
        anim.frameWidth,
        anim.frameHeight,
        facingRight ? this.position.x : canvas.width - this.position.x - this.width,
        this.position.y,
        this.width,
        this.height
      );
      c.restore();
    }

    takeDamage(amount) {
      if (!this.isAlive) return;
      this.health -= amount;
      if (this.health <= 0) {
        this.health = 0;
        this.isAlive = false;
        this.currentAnimation = 'idle';
        this.velocity.x = 0;
      } else {
        this.currentAnimation = 'hurt';
        this.frameIndex = 0;
        this.frameCount = 0;
      }
    }

    update() {
      if (!this.isAlive) return;

      const dx = character.position.x - this.position.x;
      const distance = Math.abs(dx);
      const attackDistance = 100;
      const chaseDistance = 500;

      if (distance < attackDistance) {
        this.currentAnimation = this.currentAnimation !== 'hurt' ? 'attack' : this.currentAnimation;
        this.velocity.x = 0;
        if (this.attackCooldown <= 0) {
          character.takeDamage(5); // Her saldırı döngüsünde 5 hasar
          this.attackCooldown = 60; // 1 saniye gecikme (60 FPS)
        }
      } else if (distance < chaseDistance) {
        this.currentAnimation = this.currentAnimation !== 'hurt' ? 'attack' : this.currentAnimation;
        this.velocity.x = dx > 0 ? 2 : -2;
      } else {
        this.currentAnimation = this.currentAnimation !== 'hurt' ? 'idle' : this.currentAnimation;
        this.velocity.x = 0;
      }

      if (this.attackCooldown > 0) {
        this.attackCooldown--;
      }

      const anim = this.animations[this.currentAnimation];
      this.frameCount++;
      if (this.frameCount >= 60 / anim.frameRate) {
        this.frameIndex = (this.frameIndex + 1) % anim.frameCount;
        this.frameCount = 0;
        if (this.currentAnimation === 'hurt' && this.frameIndex === 0) {
          this.currentAnimation = distance < chaseDistance ? 'attack' : 'idle';
        }
      }

      this.position.x += this.velocity.x;
      this.position.y += this.velocity.y;

      if (this.position.y + this.height + this.velocity.y >= collisionBlock.position.y) {
        this.velocity.y = 0;
        this.position.y = collisionBlock.position.y - this.height;
      } else {
        this.velocity.y += gravity;
      }

      this.draw();
    }
  }

  class CollisionBlock {
    constructor({ position, width, height }) {
      this.position = position;
      this.width = width;
      this.height = height;
    }

    draw() {
    }

    update() {
      this.draw();
    }
  }

  const character = new Character({
    position: { x: 100, y: 0 },
    animations: {
      idle: {
        image: new Image(),
        imageSrc: './img/idle.png',
        frameWidth: 64,
        frameHeight: 64,
        frameCount: 2,
        frameRate: 2,
        loaded: false,
      },
      walk: {
        image: new Image(),
        imageSrc: './img/walk.png',
        frameWidth: 64,
        frameHeight: 64,
        frameCount: 1,
        frameRate: 2,
        loaded: false,
      },
      jump: {
        image: new Image(),
        imageSrc: './img/jump.png',
        frameWidth: 64,
        frameHeight: 64,
        frameCount: 1,
        frameRate: 30,
        loaded: false,
      },
      attack: {
        image: new Image(),
        imageSrc: './img/attack.png',
        frameWidth: 64,
        frameHeight: 64,
        frameCount: 2,
        frameRate: 5,
        loaded: false,
      },
      hurt: {
        image: new Image(),
        imageSrc: './img/hurt.png',
        frameWidth: 64,
        frameHeight: 64,
        frameCount: 2,
        frameRate: 5,
        loaded: false,
      },
    },
  });

  const enemy = new Enemy({
    position: { x: canvas.width - 400, y: 0 },
    animations: {
      idle: {
        image: new Image(),
        imageSrc: './img/enemy_idle.png',
        frameWidth: 192,
        frameHeight: 192,
        frameCount: 4,
        frameRate: 2,
        loaded: false,
      },
      attack: {
        image: new Image(),
        imageSrc: './img/enemy_attack.png',
        frameWidth: 192,
        frameHeight: 192,
        frameCount: 3,
        frameRate: 5,
        loaded: false,
      },
      hurt: {
        image: new Image(),
        imageSrc: './img/enemy_hurt.png',
        frameWidth: 192,
        frameHeight: 192,
        frameCount: 3,
        frameRate: 5,
        loaded: false,
      },
    },
  });

  Object.values(character.animations).forEach(anim => {
    anim.image.src = anim.imageSrc;
    anim.image.onload = () => {
      anim.loaded = true;
    };
  });

  Object.values(enemy.animations).forEach(anim => {
    anim.image.src = anim.imageSrc;
    anim.image.onload = () => {
      anim.loaded = true;
    };
  });

  const background = new Sprite({
    position: { x: 0, y: 0 },
    imageSrc: './img/background.jpg',
  });

  const collisionBlock = new CollisionBlock({
    position: { x: 0, y: canvas.height - 180 },
    width: canvas.width,
    height: 20,
  });

  const keys = {
    d: { pressed: false },
    a: { pressed: false },
  };

  function animate() {
    window.requestAnimationFrame(animate);
    c.fillStyle = 'white';
    c.fillRect(0, 0, canvas.width, canvas.height);
    background.update();
    collisionBlock.update();
    character.update();
    enemy.update();

    // Oyuncu saldırısı için çarpışma kontrolü
    if (character.currentAnimation === 'attack' && character.isAlive && enemy.isAlive) {
      const dx = character.position.x - enemy.position.x;
      const dy = character.position.y - enemy.position.y;
      const distance = Math.sqrt(dx * dx + dy * dy);
      if (distance < character.width) {
        enemy.takeDamage(10); // Oyuncu saldırısı 10 hasar verir
      }
    }

    character.velocity.x = 0;

    if (keys.d.pressed) {
      character.velocity.x = 2;
      character.setAnimation('walk');
      character.facingRight = true;
    } else if (keys.a.pressed) {
      character.velocity.x = -2;
      character.setAnimation('walk');
      character.facingRight = false;
    }
  }
  animate();

  window.addEventListener('keydown', (event) => {
    switch (event.key) {
      case 'd':
        keys.d.pressed = true;
        break;
      case 'a':
        keys.a.pressed = true;
        break;
      case ' ':
        if (character.position.y + character.height >= collisionBlock.position.y) {
          character.velocity.y = -20;
          character.setAnimation('jump');
        }
        break;
      case 's':
        character.setAnimation('attack');
        break;
    }
  });

  window.addEventListener('keyup', (event) => {
    switch (event.key) {
      case 'd':
        keys.d.pressed = false;
        break;
      case 'a':
        keys.a.pressed = false;
        break;
    }
  });
</script>
```
-Karakter Sınıfına Bayrak Ekleyin:

-Character sınıfının yapıcı metodunda (constructor), this.attackHit = false; ekleyin:
```
class Character {
  constructor({ position, animations }) {
    this.position = position;
    this.velocity = { x: 0, y: 1 };
    this.width = 120;
    this.height = 120;
    this.animations = animations;
    this.currentAnimation = 'idle';
    this.frameIndex = 0;
    this.frameCount = 0;
    this.facingRight = true;
    this.health = 100;
    this.isAlive = true;
    this.attackHit = false; // Yeni eklenen bayrak
  }
  // ... diğer metodlar ...
}
```
-Animate Fonksiyonunda Değişiklik Yapın:

-animate fonksiyonunda, saldırı kontrolünü şu şekilde değiştirin:
```
  function animate() {
  window.requestAnimationFrame(animate);
  c.fillStyle = 'white';
  c.fillRect(0, 0, canvas.width, canvas.height);
  background.update();
  collisionBlock.update();
  character.update();
  enemy.update();

  // Saldırı mantığı
  if (character.currentAnimation === 'attack' && !character.attackHit && character.isAlive && enemy.isAlive) {
    if (
      character.position.x < enemy.position.x + enemy.width &&
      character.position.x + character.width > enemy.position.x &&
      character.position.y < enemy.position.y + enemy.height &&
      character.position.y + character.height > enemy.position.y
    ) {
      enemy.takeDamage(10);
      character.attackHit = true;
    }
  }

  character.velocity.x = 0;
  // ... diğer kodlar ...
}
```
-Karakterin Update Metodunda Bayrağı Sıfırlayın:

-Character sınıfının update metodunda, saldırı animasyonu bittiğinde (örneğin, animasyon "idle" durumuna döndüğünde) attackHit bayrağını sıfırlayın. Bu, şu şekilde yapılabilir:

```
class Character {
  // ... diğer metodlar ...
  update() {
    if (!this.isAlive) return;

    const anim = this.animations[this.currentAnimation];
    this.frameCount++;
    if (this.frameCount >= 60 / anim.frameRate) {
      this.frameIndex = (this.frameIndex + 1) % anim.frameCount;
      this.frameCount = 0;
      if ((this.currentAnimation === 'attack' || this.currentAnimation === 'jump' || this.currentAnimation === 'hurt') && this.frameIndex === 0) {
        this.currentAnimation = 'idle';
        if (this.currentAnimation === 'attack') { // Bu satır aslında gerekmeyebilir, ama emin olmak için
          this.attackHit = false;
        }
      }
    }
    // ... diğer kodlar ...
  }
  // ... diğer metodlar ...
}
```

### SORGU 2
- Arkaplanda müzik çalışmıyor bunun sebebi nedir Güncel kod bu:
```
<!DOCTYPE html>
<html>
<head>
  <style>
    body {
      background: black;
      margin: 0;
      overflow: hidden;
    }
  </style>
</head>
<body>
<canvas></canvas>
<audio id="backgroundMusic" autoplay loop>
  <source src="./audio/background_music.mp3" type="audio/mpeg">
  Your browser does not support the audio element.
</audio>
<script>
  const canvas = document.querySelector('canvas');
  const c = canvas.getContext('2d');

  // Canvas boyutunu ayarla
  canvas.width = 1920;
  canvas.height = 932;

  const gravity = 0.5;

  // Oyun dünyası boyutları
  const world = {
    width: 3840, // Arka plan genişliğinin 2 katı
    height: 932
  };

  // Kamera nesnesi
  const camera = {
    x: 0,
    y: 0,
    width: canvas.width,
    height: canvas.height,
    follow: function(target) {
      // Karakterin ekranın ortasında kalmasını sağla
      const targetX = target.position.x - this.width / 2 + target.width / 2;
      
      // Kamera sınırlarını ayarla
      this.x = Math.max(0, Math.min(targetX, world.width - this.width));
      this.y = 0; // Dikey kamera hareketi yok
    }
  };

  class Sprite {
    constructor({ position, imageSrc }) {
      this.position = position;
      this.image = new Image();
      this.image.src = imageSrc;
      this.loaded = false;
      this.image.onload = () => {
        this.loaded = true;
      };
    }

    draw() {
      if (!this.image || !this.loaded) return;
      // Arka planı kameraya göre çiz
      c.drawImage(
        this.image,
        -camera.x, // Kamera x pozisyonuna göre kaydır
        0,
        world.width, // Arka plan dünya genişliği kadar
        canvas.height
      );
    }

    update() {
      this.draw();
    }
  }

  class Character {
    constructor({ position, animations }) {
      this.position = position;
      this.velocity = { x: 0, y: 1 };
      this.width = 120;
      this.height = 120;
      this.animations = animations;
      this.currentAnimation = 'idle';
      this.frameIndex = 0;
      this.frameCount = 0;
      this.facingRight = true;
      this.health = 100;
      this.isAlive = true;
      this.attackHit = false;
    }

    draw() {
      if (!this.isAlive) return;
      const anim = this.animations[this.currentAnimation];
  if (!anim.image || !anim.loaded) return;

  c.save();
  if (!this.facingRight) {
    c.scale(-1, 1);
    c.translate(-canvas.width, 0);
  }
  
  const drawX = this.facingRight ? 
               this.position.x - camera.x : 
               canvas.width - (this.position.x - camera.x) - this.width;
  
  c.drawImage(
    anim.image,
    this.frameIndex * anim.frameWidth,
    0,
    anim.frameWidth,
    anim.frameHeight,
    drawX,
    this.position.y - camera.y,
    this.width,
    this.height
  );
  c.restore();
}

takeDamage(amount) {
  if (!this.isAlive) return;
  this.health -= amount;
  if (this.health <= 0) {
    this.health = 0;
    this.isAlive = false;
    this.currentAnimation = 'idle';
    this.velocity.x = 0;
  } else {
    this.setAnimation('hurt');
  }
}

update() {
  if (!this.isAlive) return;

  const anim = this.animations[this.currentAnimation];
  this.frameCount++;
  if (this.frameCount >= 60 / anim.frameRate) {
    this.frameIndex = (this.frameIndex + 1) % anim.frameCount;
    this.frameCount = 0;
    if ((this.currentAnimation === 'attack' || this.currentAnimation === 'jump' || this.currentAnimation === 'hurt') && this.frameIndex === 0) {
      if (this.currentAnimation === 'attack') { 
        this.attackHit = false;
      }
      this.currentAnimation = 'idle';
    }
  }

  this.position.x += this.velocity.x;
  this.position.y += this.velocity.y;

  // Dünya sınırlarını kontrol et
  this.position.x = Math.max(0, Math.min(this.position.x, world.width - this.width));

  if (this.position.y + this.height + this.velocity.y >= canvas.height - 180) {
    this.velocity.y = 0;
    this.position.y = canvas.height - 180 - this.height;
    if (this.currentAnimation === 'jump') {
      this.currentAnimation = 'idle';
    }
  } else {
    this.velocity.y += gravity;
  }

  this.draw();
}

setAnimation(animation) {
  if (this.currentAnimation !== animation) {
    this.currentAnimation = animation;
    this.frameIndex = 0;
    this.frameCount = 0;
  }
}
}

class Enemy {
constructor({ position, animations }) {
  this.position = position;
  this.velocity = { x: 0, y: 0.3 };
  this.width = 300;
  this.height = 300;
  this.animations = animations;
  this.currentAnimation = 'idle';
  this.frameIndex = 0;
  this.frameCount = 0;
  this.health = 100;
  this.isAlive = true;
  this.attackCooldown = 1000;
}

draw() {
  if (!this.isAlive) return;
  const anim = this.animations[this.currentAnimation];
  if (!anim.image || !anim.loaded) return;
  
  c.save();
  const facingRight = this.position.x > character.position.x;
  if (!facingRight) {
    c.scale(-1, 1);
    c.translate(-canvas.width, 0);
  }
  
  const drawX = facingRight ? 
               this.position.x - camera.x : 
               canvas.width - (this.position.x - camera.x) - this.width;
  
  c.drawImage(
    anim.image,
    this.frameIndex * anim.frameWidth,
    0,
    anim.frameWidth,
    anim.frameHeight,
    drawX,
    this.position.y - camera.y,
    this.width,
    this.height
  );
  c.restore();
}

takeDamage(amount) {
  if (!this.isAlive) return;
  this.health -= amount;
  if (this.health <= 0) {
    this.health = 0;
    this.isAlive = false;
    this.currentAnimation = 'idle';
    this.velocity.x = 0;
  } else {
    this.currentAnimation = 'hurt';
    this.frameIndex = 0;
    this.frameCount = 0;
  }
}

update() {
  if (!this.isAlive) return;

  const dx = character.position.x - this.position.x;
  const distance = Math.abs(dx);
  const attackDistance = 100;
  const chaseDistance = 500;

  if (distance < attackDistance) {
    this.currentAnimation = this.currentAnimation !== 'hurt' ? 'attack' : this.currentAnimation;
    this.velocity.x = 0;
    if (this.attackCooldown <= 0) {
      character.takeDamage(5);
      this.attackCooldown = 60;
    }
  } else if (distance < chaseDistance) {
    this.currentAnimation = this.currentAnimation !== 'hurt' ? 'attack' : this.currentAnimation;
    this.velocity.x = dx > 0 ? 2 : -2;
  } else {
    this.currentAnimation = this.currentAnimation !== 'hurt' ? 'idle' : this.currentAnimation;
    this.velocity.x = 0;
  }

  if (this.attackCooldown > 0) {
    this.attackCooldown--;
  }

  const anim = this.animations[this.currentAnimation];
  this.frameCount++;
  if (this.frameCount >= 60 / anim.frameRate) {
    this.frameIndex = (this.frameIndex + 1) % anim.frameCount;
    this.frameCount = 0;
    if (this.currentAnimation === 'hurt' && this.frameIndex === 0) {
      this.currentAnimation = distance < chaseDistance ? 'attack' : 'idle';
    }
  }

  this.position.x += this.velocity.x;
  this.position.y += this.velocity.y;

  // Düşmanın dünya sınırlarını kontrol et
  this.position.x = Math.max(0, Math.min(this.position.x, world.width - this.width));

  if (this.position.y + this.height + this.velocity.y >= canvas.height - 180) {
    this.velocity.y = 0;
    this.position.y = canvas.height - 180 - this.height;
  } else {
    this.velocity.y += gravity;
  }

  this.draw();
}
}

class CollisionBlock {
constructor({ position, width, height }) {
  this.position = position;
  this.width = width;
  this.height = height;
}

draw() {
  
}

update() {
  this.draw();
}
}

// Karakter oluşturma
const character = new Character({
position: { x: 100, y: 0 },
animations: {
  idle: {
    image: new Image(),
    imageSrc: './img/idle.png',
    frameWidth: 64,
    frameHeight: 64,
    frameCount: 2,
    frameRate: 2,
    loaded: false,
  },
  walk: {
    image: new Image(),
    imageSrc: './img/walk.png',
    frameWidth: 64,
    frameHeight: 64,
    frameCount: 1,
    frameRate: 2,
    loaded: false,
  },
  jump: {
    image: new Image(),
    imageSrc: './img/jump.png',
    frameWidth: 64,
    frameHeight: 64,
    frameCount: 1,
    frameRate: 30,
    loaded: false,
  },
  attack: {
    image: new Image(),
    imageSrc: './img/attack.png',
    frameWidth: 64,
    frameHeight: 64,
    frameCount: 2,
    frameRate: 5,
    loaded: false,
  },
  hurt: {
    image: new Image(),
    imageSrc: './img/hurt.png',
    frameWidth: 64,
    frameHeight: 64,
    frameCount: 2,
    frameRate: 5,
    loaded: false,
  },
},
});

// Düşman oluşturma
const enemy = new Enemy({
position: { x: 2000, y:0 },
animations: {
  idle: {
    image: new Image(),
    imageSrc: './img/enemy_idle.png',
    frameWidth: 192,
    frameHeight: 192,
    frameCount: 4,
    frameRate: 2,
    loaded: false,
  },
  attack: {
    image: new Image(),
    imageSrc: './img/enemy_attack.png',
    frameWidth: 192,
    frameHeight: 192,
    frameCount: 3,
    frameRate: 5,
    loaded: false,
  },
  hurt: {
    image: new Image(),
    imageSrc: './img/enemy_hurt.png',
    frameWidth: 192,
    frameHeight: 192,
    frameCount: 3,
    frameRate: 5,
    loaded: false,
  },
},
});

// Animasyon resimlerini yükle
Object.values(character.animations).forEach(anim => {
anim.image.src = anim.imageSrc;
anim.image.onload = () => {
  anim.loaded = true;
};
});

Object.values(enemy.animations).forEach(anim => {
anim.image.src = anim.imageSrc;
anim.image.onload = () => {
  anim.loaded = true;
};
});

// Arka plan oluşturma
const background = new Sprite({
position: { x: 0, y: 0 },
imageSrc: './img/background.jpg',
});

const collisionBlock = new CollisionBlock({
position: { x: 0, y: canvas.height - 180 },
width: world.width,
height: 20,
});

// Klavye kontrolleri
const keys = {
d: { pressed: false },
a: { pressed: false },
};

function animate() {
window.requestAnimationFrame(animate);
c.fillStyle = 'white';
c.fillRect(0, 0, canvas.width, canvas.height);

// Arka planı sabit çiz
background.update();

// Kamerayı güncelle
camera.follow(character);

// Diğer nesneleri çiz
collisionBlock.update();
character.update();
enemy.update();

// Oyuncu saldırısı için çarpışma kontrolü
if (character.currentAnimation === 'attack' && !character.attackHit && character.isAlive && enemy.isAlive) {
  if (
    character.position.x < enemy.position.x + enemy.width &&
    character.position.x + character.width > enemy.position.x &&
    character.position.y < enemy.position.y + enemy.height &&
    character.position.y + character.height > enemy.position.y
  ) {
    enemy.takeDamage(10);
    character.attackHit = true;
  }
}

character.velocity.x = 0;

if (keys.d.pressed) {
  character.velocity.x = 5;
  character.setAnimation('walk');
  character.facingRight = true;
} else if (keys.a.pressed) {
  character.velocity.x = -5;
  character.setAnimation('walk');
  character.facingRight = false;
}
}

animate();

// Klavye olayları
window.addEventListener('keydown', (event) => {
switch (event.key) {
  case 'd':
    keys.d.pressed = true;
    break;
  case 'a':
    keys.a.pressed = true;
    break;
  case ' ':
    if (character.position.y + character.height >= canvas.height - 180) {
      character.velocity.y = -20;
      character.setAnimation('jump');
    }
    break;
  case 's':
    character.setAnimation('attack');
    break;
}
});

window.addEventListener('keyup', (event) => {
switch (event.key) {
  case 'd':
    keys.d.pressed = false;
    if (!keys.a.pressed) {
      character.setAnimation('idle');
    }
    break;
  case 'a':
    keys.a.pressed = false;
    if (!keys.d.pressed) {
      character.setAnimation('idle');
    }
    break;
}
});

// Pencere boyutu değiştiğinde canvas'ı yeniden boyutlandır
window.addEventListener('resize', () => {
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
camera.width = canvas.width;
camera.height = canvas.height;
});
</script>
</body>
</html>
```
-Neden Müziğiniz Çalışmıyor?
-Modern tarayıcılar, kullanıcı izni olmadan sesli içeriklerin otomatik çalınmasını engelleyen politikalar uygular. Bu, özellikle Chrome gibi tarayıcılarda sık görülür ve müziğinizin otomatik başlamamasının ana nedeni olabilir. Ayrıca, müzik dosyanızın yolu (./audio/background_music.mp3) doğru olmayabilir veya dosya formatı (MP3) tarayıcınız tarafından desteklenmeyebilir.

-Çözüm Önerileri
-Kullanıcı Etkileşimi: Bir düğme ekleyerek kullanıcının müziği manuel başlatmasını sağlayabilirsiniz. Örneğin:
```
<button onclick="document.getElementById('backgroundMusic').play()">Müziği Başlat</button>
```
-Sessiz Başlatma: Müziği başlangıçta sessiz (muted) olarak başlatıp, kullanıcı etkileşimiyle sesi açabilirsiniz:
```
<audio id="backgroundMusic" autoplay loop muted>
  <source src="./audio/background_music.mp3" type="audio/mpeg">
```
-Ardından, JavaScript ile sesi açabilirsiniz:
```
  document.addEventListener('click', function() {
  document.getElementById('backgroundMusic').muted = false;
});
```
-Dosya Kontrolü: Müzik dosyasının yolunun doğru olduğundan ve dosyanın tarayıcınız tarafından desteklenen bir formatta (örneğin MP3, OGG) olduğundan emin olun. Farklı formatlar için:
```
<audio id="backgroundMusic" autoplay loop>
  <source src="./audio/background_music.mp3" type="audio/mpeg">
  <source src="./audio/background_music.ogg" type="audio/ogg">
```
