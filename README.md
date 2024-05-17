# anki-videoflashcards-note-type

## Front Template

```js
<script>
// --- Options ---
var config = {
  'auto-play-on-backside': 'video', // 'video' or 'audio'
  'tap-on-screen-will-play-video': false, // true or false
  'display-meaning-as-hint': true, // true or false
  'tap-on-screen-will-reveal-hint': false,
  'auto-reveal-meaning-on-sound-end': false,
  'key-to-reveal-meaning': 'KeyG', // 'g' (only english layout) or 'KeyG' (any keyboard layout)
}

// On mobile, the video will take up the full width of the screen.
// To make it look the same on mobile as on desktop, remove 'youtube' from 'class="player youtube"' in the 'div' tag below.
</script>

<div class="player">
  <video poster="{{text:Id}}.jpg" playsinline onclick="togglePlayPause(); return false;" controlsList="nodownload" disablepictureinpicture disableRemotePlayback>
    <source src="{{text:Id}}.mp4" type="video/mp4">
    <source src="{{text:Id}}.webm" type="video/webm">
  </video>
</div>

<!-- Replay butonu -->
<a class="replay-button video" href="#" onclick="restartVideo(); return false;">Replay from Start</a>
   
<div hidden>{{Snapshot}}</div>

<audio id="audio" controls hidden></audio>

<script>
var isVideoError = false;

function playAudio() {
  stopVideo();
  let audio = document.getElementById('audio');
  audio.currentTime = 0;
  audio.play();
}

(() => {
  let audio = document.getElementById('audio');
  audio.src = `{{text:Id}}.mp3`;

  let video = document.querySelector('video');
  let source = video.querySelector("source:last-child");
  source.addEventListener('error', () => {
    isVideoError = true;
    playAudio();
  }, true);
})();
</script>

<script>
var isTextSelected = false;

function tapAction(event) {
  if (isTextSelected) return;
  if (!window.getSelection().isCollapsed) return;
  if (event.target.tagName == 'A') return;
  if (config['tap-on-screen-will-reveal-hint']) {
    let hint = document.querySelector('a.hint');
    if (hint) {
      showMeaning();
      return;
    }
  }
  togglePlayPause();
}

function togglePlayPause() {
  stopAudio();
  let video = document.querySelector('video');
  if (isVideoError) {
    return playAudio();
  }
  if (video.paused) {
    video.play();
  } else {
    video.pause();
  }
}

function restartVideo() {
  stopAudio();
  let video = document.querySelector('video');
  if (isVideoError) {
    return playAudio();
  }
  video.currentTime = 0;
  video.play();
}

function replaySound(event) {
  if (event.key != 'r' && event.code != 'KeyR') return;
  togglePlayPause();
}

if (!document.body.hasAttribute('js-replay-button-handler')) {
  document.body.setAttribute('js-replay-button-handler', '');
  document.addEventListener('keyup', replaySound);
}

if (config['tap-on-screen-will-play-video'] && !document.body.hasAttribute('js-tap-on-screen-handler')) {
  document.body.setAttribute('js-tap-on-screen-handler', '');
  document.addEventListener('click', tapAction);
  document.addEventListener('mousedown', (event) => {
    isTextSelected = !window.getSelection().isCollapsed;
  });
}
</script>

<script>
function isPlaying(element) {
  return !!(element.currentTime > 0 && !element.paused && !element.ended && element.readyState > 2);
}

function stopAudio() {
  let audio = document.querySelector('audio');
  stopPlayer(audio);
}

function stopVideo() {
  let video = document.querySelector('video');
  stopPlayer(video);
}

function stopPlayer(element) {
  if (element && isPlaying(element)) {
    element.pause();
  }
}
</script>

<script>
function playSound() {
  if (document.querySelector('hr#answer') && document.querySelector('audio') && config['auto-play-on-backside'] == 'audio') {
    playAudio();
  } else {
    togglePlayPause();
  }
}

function autoplay() {
  if (document.documentElement.classList.contains("android")) {
    playSound();
  } else {
    setTimeout(() => {
      playSound();
    }, 25);
  }
}

if (typeof onShownHook !== 'undefined') {
  onShownHook.push(autoplay);
} else {
  autoplay();
}
</script>
```
### Front Template
- **config:** Çeşitli ayarlar için kullanılan bir yapı. Videoların otomatik oynatılması, ipuçlarının gösterilmesi ve klavye kısayolları gibi özellikleri içerir.
- **Video Player:** `<video>` elementi, `poster` özelliği ile bir ön izleme görüntüsü belirler ve `source` etiketleri ile farklı video formatlarını destekler. `onclick` olayına `togglePlayPause` işlevi bağlanmış.
- **Replay Butonu:** `Replay from Start` işlevi için bir buton tanımlanmış.
- **Audio Elementi:** `audio` elementi, `id="audio"` ile tanımlanmış ve başlangıçta gizlenmiş.
- **JavaScript Kodları:**
  - `playAudio`: Videoyu durdurup sesi oynatır.
  - `togglePlayPause`: Video oynatılıyorsa durdurur, duruyorsa oynatır. Eğer video yüklenemiyorsa ses oynatır.
  - `restartVideo`: Videoyu baştan başlatır.
  - `replaySound`: Klavye kısayolu ile videoyu durdurup başlatır.
  - `isPlaying`: Bir medya elementinin oynayıp oynamadığını kontrol eder.
  - `stopAudio` ve `stopVideo`: Ses ve video elementlerini durdurur.
  - `playSound` ve `autoplay`: Medya içeriğini otomatik olarak oynatır.
  - `tapAction`: Ekrana dokunma eylemini yönetir, video oynatmayı veya duraklatmayı kontrol eder.

## Back Template

```js
{{FrontSide}}

<hr id=answer>

{{#Expression}}
<div class="expression">{{edit:Expression}}</div>
{{/Expression}}

{{#Meaning}}
<a class="hint" href="#" onclick="this.style.display='none';document.getElementById('hint4753594160').style.display='block';return false;">...</a><div id="hint4753594160" class="meaning" style="display:none">{{edit:Meaning}}</div>
{{/Meaning}}

<a class="replay-button audio" href="#" onclick="playAudio();return false;"></a>

{{#Notes}}
<div><div class="notes">{{edit:Notes}}</div></div>
{{/Notes}}

<script>
function showMeaning() {
  let hint = document.querySelector('a.hint');
  if (hint) {
    hint.click();
    hint.remove();
  }
}

(function () {
  if (!config['display-meaning-as-hint']) {
    let hint = document.querySelector('a.hint');
    if (hint) hint.click();
  }

  if (config['auto-reveal-meaning-on-sound-end']) {
    let sound = document.querySelector('video');
    if (config['auto-play-on-backside'] == 'audio') {
      sound = document.querySelector('audio');
    }
    sound.addEventListener("ended", () => {
      setTimeout(showMeaning, 500);
    });
  }

  if (config['key-to-reveal-meaning']) {
    document.addEventListener('keyup', () => {
      if (![event.key, event.code].includes(config['key-to-reveal-meaning'])) return;
      showMeaning();
    });
  }
})();
</script>
```

### Back Template
- **FrontSide:** Kartın ön yüzünün içeriğini gösterir.
- **Expression ve Meaning:** Kullanıcıya gösterilecek ifade ve anlam bilgilerini içerir.
- **Replay Butonu:** Ses dosyasını yeniden çalmak için bir buton ekler.
- **JavaScript Kodları:**
  - `showMeaning`: İpucu gösterir.
  - Anlamın otomatik olarak gösterilmesi ve klavye kısayolu ile gösterilmesi için olay dinleyicileri ekler.

## Styling

```css
.card {
 font-family: arial;
 font-size: 20px;
 text-align: center;
 color: var(--primary-color);
 background-color: var(--background-color);
}

.card1, .mobile .card1 #content {
 margin: 0;
}

#qa {
 margin: 20px;
}

.mobile #qa {
 margin: 10px;
}

.mobile .player.youtube {
 margin: -10px -10px 0 -10px;
 display: block;
 padding: 0px;
}

.mobile .player.youtube video {
 border-radius: 0;
 max-height: 100vh;
}

.mobile .player.youtube, .mobile .player.youtube video {
 background: #000;
}

.mobile .nightMode .player.youtube, .mobile .nightMode .player.youtube video {
 background: #000;
}

:root {
 --primary-color: rgb(0 0 0 / 97%);
 --secondary-color: rgb(0 0 0 / 65%);
 --meaning-color: #000080;
 --background-color: white;
}

:root .nightMode {
 --primary-color: white;
 --secondary-color: #aaa;
 --meaning-color: #9bc0dd;
 --background-color: #2f2f31;
}

.expression {
 font-size: 18px;
 line-height: 124%;
 color: var(--primary-color);
}

.meaning, a.hint {
 display: block;
 font-size: 18px;
 line-height: 120%;
 margin-bottom: 3px;
 margin-top: 11px;
}

.meaning {
 color: var(--meaning-color);
}

a.hint {
 color: #777;
 text-decoration: none;
}

.notes {
 color: var(--secondary-color);
 display: inline-block;
 text-align: left;
 margin: 10px auto;
}

img {
 max-width: 100%;
 height: auto;
}

.player {

}

video {
  display: inline-block;    /* Videoyu inline olarak hizala ve blok eleman gibi davranmasını sağla */
  vertical-align: middle;   /* Videoyu dikey olarak ortala */
  margin: 0 auto 0;      /* Üstten 20px boşluk bırak ve alttan ve yandan boşluk bırakma */
  max-width: 100%;           /* Maksimum genişlik olarak kapsayıcının genişliğine eşit yap */
  max-height: 90vh;         /* Maksimum yükseklik olarak ekranın %90'ına eşit yap */
  border-radius: 5px;       /* Kenarları yuvarlaklaştır, yarıçapı 5px */
}

video::-webkit-media-controls-fullscreen-button {
 display: none;
}

.replay-button {
 width: 34px;
 padding: 5px;
 user-select: none;
 content: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' class='playImage' viewBox='0 0 64 64' width='40px' height='40px' version='1.1'%3E%3Ccircle fill='%23fff' stroke='%23414141' cx='32' cy='32' r='29'%3E%3C/circle%3E%3Cpath fill='%23414141' d='M56.502,32.301l-37.502,20.101l0.329,-40.804l37.173,20.703Z'%3E%3C/path%3E%3C/svg%3E");
}

.replay-button.video {
 margin-top: 9px;
}

.android .replay-button.video {
 margin-top: 10px;
}

.replay-button.audio {
 margin-top: 5px;
}

hr#answer {
 margin-top: 7px;
 margin-bottom: 16px;
 height: 1px;
 background-color: #9a9a9a;
 border: none;
}

.nightMode .replay-button {
 filter: invert(85%);
}

/* CARD 2 STYLING */

.card2 .meaning {
 color: var(--primary-color);
 font-size: 20px;
}

.card2 .expression {
 font-size: 20px;
}

.card2 hr#answer {
 margin-top: 12.5px;
 margin-bottom: 12.5px;
}

.card2 .media {
 margin-top: 2.5px;
}

.card2 .snapshot img {
 display: inline-block;
 vertical-align: middle;
 margin: 0 auto;
 max-width: 100%;
 max-height: 90vh;
 border-radius: 3px;
 max-height: 260px;
 margin-top: 10px;
}

.replay-button.audio {
    display: none;
}
```

### Styling
- **General Card Style:** Kartın genel stilini belirler. Yazı tipi, boyut, renkler ve hizalama gibi özellikler içerir.
- **Video Player Style:** Video elementinin stilini belirler. Dikey hizalama, kenar boşlukları, maksimum genişlik ve yükseklik gibi özellikler içerir.
- **Replay Button Style:** Yeniden oynatma butonlarının stilini belirler.
- **Night Mode:** Gece modunda renklerin nasıl değişeceğini belirler.