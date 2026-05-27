# DEADZONE — Böngészős 3D FPS Játék

Egyetlen HTML fájlban megvalósított, teljesen működő first-person lövöldözős játék.  
Külső függőség mindössze egy: **Three.js r128** (CDN-ről töltődik be).

---

## Gyors indítás

1. Töltsd le a `deadzone_fps.html` fájlt.
2. Nyisd meg egy modern böngészőben (Chrome, Firefox, Edge).
3. Kattints a **JÁTÉK INDÍTÁSA** gombra.
4. A játékterületre kattintva az egér zárolódik — ekkor indul az irányítás.

> **Megjegyzés:** Fájlrendszerből (`file://`) megnyitva egyes böngészők blokkolhatják az AudioContext-et. Ha nincs hang, egy kattintás általában feloldja. Lokális webszerver (pl. `npx serve .`) használata javasolt a legjobb élményért.

---

## Irányítás

| Gomb | Funkció |
|------|---------|
| `W A S D` | Mozgás |
| `SHIFT` | Sprint (gyorsabb mozgás, szétszórtabb lövés) |
| `SPACE` | Ugrás |
| `Bal egérgomb` | Lövés (tartva: automata tűz) |
| `R` | Újratöltés |
| `Egér` | Körbenézés |
| `ESC` | Egér feloldása / szünet |

---

## Játékmenet

- **Hullámok:** Az ellenfelek hullámokban spawnolnak. Az első hullámban 5 ellenfél van, minden további hullámban +3-mal nő a számuk.
- **Nehézség:** Minden hullámmal nő az ellenfelek életereje, sebessége és pontossága.
- **Pontrendszer:** Ellenfél ölése = **10 pont**, fejlövéssel = **25 pont** (azonnali ölés).
- **HP rendszer:** A játékos 100 HP-val indul. Az ellenfelek lövedékei és közelharci érintése sebzik. 0 HP → Game Over.
- **Lőszer:** Végtelen tartalék, de a tár 30 töltényes — ürülés után újra kell tölteni.

---

## Funkciók

### Grafika & Fizika
- **Three.js WebGL** renderelés, árnyékokkal és exponenciális köddel
- **First-person kamera**, egér-zárolással (Pointer Lock API)
- Egyszerű AABB **ütközésdetekció**: játékos, ellenfelek és lövedékek sem mennek át a fedezékdobozokon
- **Gravitáció + ugrás** fizika (velY, talajon-ellenőrzés)
- Fegyver **bob animáció** mozgáskor, **recoil** lövéskor, **idle lélegzés** állva

### Ellenfelek (AI)
- Folyamatosan a játékos felé navigálnak
- Megkerülik (nem mennek át) a fedezékdobozokon
- Lőnek a játékosra (pontosság hullámonként nő)
- Közelharci sebzés ha elérnek
- Fejük (`userData.isHead`) külön hitbox — fejlövés = azonnali halál
- HP-sáv mindig látható felettük

### Hangrendszer (Web Audio API)
Valódi hangfájl nélkül, proceduálisan generált effektek:

| Esemény | Hang |
|---------|------|
| Lövés | Zajburst + lowpass szűrő |
| Ellenfél találat | Sawtooth sweep |
| Fejlövés | Kettős square hangeffekt |
| Üres tár | Kattanás |
| Újratöltés | Háromszoros mechanikus kattintás |
| Ugrás | Rövid sine glissando |
| Landolás | Tompított zajütés |
| Hullám kezdete | Sawtooth fanfár |

### Vizuális effektek
- **Részecske-rendszer** — találatkor, ellenfél halálakor szétrepülő kockák
- **Muzzle flash** — fényforrás (`PointLight`) + képernyő-villanás
- **Damage flash** — piros vignett találatkor
- **Low HP border** — pulzáló piros keret 30 HP alatt
- **Hitmarker** — fehér jelző (sárga + „HEADSHOT" felirat fejlövésnél)

---

## Fájlstruktúra

A projekt egyetlen fájlból áll:

```
deadzone_fps.html
├── <style>        — Teljes CSS (HUD, menük, overlay-ek)
├── <body>         — HTML struktúra (menü, HUD, game over képernyő)
├── Three.js CDN   — https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js
└── <script>       — Teljes játéklogika (~900 sor JS), szekciók:
    ├── Renderer & Scene     — Three.js beállítás, kamera, renderer
    ├── Audio                — Web Audio API, összes hangeffekt
    ├── Game State           — Játékállapot változók
    ├── Player               — HP, lőszer, tűz, sprint, ugrás adatok
    ├── Input                — Billentyűzet, egér, pointer lock
    ├── Physics              — Vízszintes és függőleges sebesség
    ├── Entity Arrays        — Ellenfelek, lövedékek, részecskék tömbök
    ├── Weapon Model         — Fegyver 3D geometria (camera child)
    ├── Environment          — Pálya: padló, falak, fedezékdobozok, fények
    ├── Enemy Factory        — Ellenfél mesh + collider + hitbox létrehozása
    ├── Wave Management      — Hullám spawn logika
    ├── Player Shoot         — Raycast + károkozás + tracer bullet
    ├── Enemy Bullets        — Ellenfél lövés + lövedék mozgás
    ├── Enemy AI             — Mozgás, célzás, tüzelés, HP-sáv frissítés
    ├── Bullet Update        — Lövedékek mozgása, ütközés fedezékkel/játékossal
    ├── Particles            — Részecske spawn + gravitáció + eltávolítás
    ├── Collision            — AABB játékos- és ellenfélfedezék-ütközés
    ├── Muzzle Flash         — Fényeffekt + képernyővillanás
    ├── Player Damage        — HP csökkentés, damage flash, game over trigger
    ├── Reload               — Újratöltés időzítő + töltőcsík UI
    ├── Player Movement      — WASD + sprint + ugrás + fegyverbob
    ├── Wave Transition      — Hullámok közötti szünet és következő hullám indítása
    ├── UI Helpers           — HUD frissítő függvények
    ├── Game Over            — Pointer unlock, végső statisztika kijelzés
    ├── Input Setup          — Event listener-ek regisztrálása
    ├── Start Game           — Reset + első hullám spawn
    └── Main Loop            — requestAnimationFrame, delta time, FPS számláló
```

---

## Továbbfejlesztési lehetőségek

### Könnyű változtatások
- **Érzékenység** — `sens = 0.0018` sor a `mousemove` handlerben
- **Tűzgyorsaság** — `PL.fireRate = 0.11` (másodperc/lövés)
- **Ugrásmagasság** — `velY = 7.2` a Space kezelőjében
- **Gravitáció ereje** — `GRAVITY = -22` az `updatePlayer`-ben
- **Ellenfélek száma** — `5 + (wave-1)*3` a `doSpawnWave`-ben
- **Lőszertár mérete** — `maxAmmo: 30` a `PL` objektumban

### Közepes fejlesztések
- **Több fegyvertípus** — shotgun (több raycast), sniper (zoom + egy lövés)
- **Minimap** — 2D canvas overlay a jelenlegi pozíciókkal
- **Ellenfél típusok** — gyors/gyenge, lassú/erős, távolsági/közeli
- **Power-up-ok** — életerő-csomag, lőszerpótlás spawnolása
- **Pontszám mentés** — `localStorage` alapú toplista

### Nagyobb fejlesztések
- **Pathfinding** — navmesh vagy A* algoritmus, hogy az ellenfelek intelligensen kerüljék meg az akadályokat
- **Több pálya** — különböző arena-k betöltése game over után
- **Multiplayerr** — WebSocket alapú hálózati szinkronizáció
- **Modell betöltés** — Three.js `GLTFLoader`-rel részletesebb 3D modellek CDN-ről

---

## Technológiák

| Technológia | Verzió | Felhasználás |
|-------------|--------|--------------|
| HTML5 | — | Struktúra, Canvas |
| CSS3 | — | HUD, menük, animációk |
| JavaScript (ES6+) | — | Teljes játéklogika |
| Three.js | r128 | 3D renderelés, geometria, fények |
| Web Audio API | — | Procedurális hangeffektek |
| Pointer Lock API | — | Egér-zárolás FPS nézethez |

---

## Böngésző-kompatibilitás

| Böngésző | Támogatás |
|----------|-----------|
| Chrome 80+ | ✅ Teljes |
| Firefox 75+ | ✅ Teljes |
| Edge 80+ | ✅ Teljes |
| Safari 14+ | ⚠️ Hang korlátozottan (AudioContext gesztus-függő) |
| Mobil | ❌ Pointer Lock hiánya miatt nem támogatott |

---

*Projekt mérete: ~1 HTML fájl, ~1400 sor kód, külső függőség: 0 (CDN kivételével).*
