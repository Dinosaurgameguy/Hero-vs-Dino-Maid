# Hero-vs-Dino-Maid
Its a retro game based on the 1989 era where a 2d hero fights a dino.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>HERO VS DINO MAID</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=VT323&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-void: #090612;
            --neon-pink: #ff2a85;
            --neon-cyan: #00f0ff;
            --neon-gold: #ffd700;
            --neon-green: #39ff14;
        }

        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-void);
            color: #ffffff;
            font-family: 'Press Start 2P', monospace;
            overflow: hidden;
            height: 100vh;
            width: 100vw;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        } /* Retro CRT Scanline Overlay */
        .crt-overlay {
            position: fixed;
            inset: 0;
            pointer-events: none;
            z-index: 50;
            background: linear-gradient(
                rgba(18, 16, 16, 0) 50%,
                rgba(0, 0, 0, 0.25) 50%
            ), linear-gradient(
                90deg,
                rgba(255, 0, 0, 0.03),
                rgba(0, 255, 0, 0.01),
                rgba(0, 0, 255, 0.03)
            );
            background-size: 100% 3px, 6px 100%;
        }
 #arcade-cabinet {
            position: relative;
            width: 100%;
            max-width: 960px;
            aspect-ratio: 16 / 9;
            max-height: 82vh;
            background: #120c24;
            border: 4px solid #3c2a6b;
            border-radius: 14px;
            box-shadow: 0 0 35px rgba(255, 42, 133, 0.35), inset 0 0 25px rgba(0, 0, 0, 0.85);
            overflow: hidden;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            image-rendering: pixelated;
            image-rendering: crisp-edges;
        }

        .pixel-btn {
            font-family: 'Press Start 2P', monospace;
            border: 3px solid #ffffff;
            box-shadow: 0 4px 0 #000000, 0 0 15px rgba(255, 42, 133, 0.5);
            transition: transform 0.08s ease, box-shadow 0.08s ease;
            cursor: pointer;
        }.pixel-btn:active {
            transform: translateY(3px);
            box-shadow: 0 1px 0 #000000;
        }

        .neon-text-pink {
            text-shadow: 2px 2px 0 #000, 0 0 12px #ff2a85;
        }

        .neon-text-cyan {
            text-shadow: 2px 2px 0 #000, 0 0 10px #00f0ff;
        }

        .neon-text-gold {
            text-shadow: 2px 2px 0 #000, 0 0 10px #ffd700;
        }

        /* Mobile Touch Controls */
        #mobile-controls {
            display: none;
        }

        @media (max-width: 820px), (pointer: coarse) {
            #mobile-controls {
                display: flex;
            } #arcade-cabinet {
                max-height: 68vh;
            }
        }

        .touch-pad-btn {
            background: rgba(34, 22, 66, 0.85);
            border: 3px solid #00f0ff;
            border-radius: 12px;
            color: #fff;
            font-size: 14px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 0 #000;
        }

        .touch-pad-btn:active {
            background: rgba(255, 42, 133, 0.85);
            transform: translateY(2px);
        }

        @keyframes pulseGlow {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.04); }
        }

        .animate-pulse-slow {
            animation: pulseGlow 1.8s infinite ease-in-out;
        } </style>
</head>
<body>
    <div class="crt-overlay"></div>

    <!-- Main Arcade Screen Container -->
    <div id="arcade-cabinet">
        <canvas id="gameCanvas"></canvas>

        <!-- Top Arcade HUD Overlay -->
        <div id="hud-overlay" class="hidden absolute top-0 left-0 right-0 p-2 sm:p-3 flex flex-col gap-1.5 pointer-events-none z-10">
            <div class="flex justify-between items-start gap-2 text-[9px] sm:text-xs">
                <!-- Player Stats -->
                <div class="bg-black/75 border-2 border-cyan-400 rounded-lg p-1.5 sm:p-2 w-40 sm:w-56">
                    <div class="flex justify-between text-cyan-300 mb-1">
                        <span>HERO</span>
                        <span id="hud-player-hp-text">100/100</span>
                    </div>
                    <div class="w-full h-2.5 bg-gray-900 border border-gray-600 rounded overflow-hidden">
                        <div id="hud-player-hp-bar" class="h-full bg-gradient-to-r from-green-400 to-cyan-400 transition-all duration-150" style="width: 100%;"></div>
                    </div><div class="w-full h-2.5 bg-gray-900 border border-gray-600 rounded overflow-hidden">
                        <div id="hud-player-hp-bar" class="h-full bg-gradient-to-r from-green-400 to-cyan-400 transition-all duration-150" style="width: 100%;"></div>
                    </div>
                    <div id="hud-buff-tag" class="mt-1 text-[8px] sm:text-[9px] text-yellow-300 truncate">
                        WEAPON: STANDARD
                    </div>
                    <div id="hud-power-tag" class="mt-0.5 text-[7px] sm:text-[8px] text-pink-300 truncate">
                        POWER: NONE (&lt;10% CHANCE)
                    </div>
                </div> <!-- Center 15s Krillion Countdown & Score -->
                <div class="bg-black/80 border-2 border-yellow-400 rounded-lg px-2.5 py-1.5 text-center">
                    <div class="text-[8px] sm:text-[9px] text-yellow-300">KRILLION QUIZ IN</div>
                    <div id="hud-krillion-timer" class="text-sm sm:text-lg text-white neon-text-gold mt-0.5">15.0s</div>
                    <div id="hud-score" class="text-[8px] text-pink-400 mt-0.5">SCORE: 0</div>
                </div>

                <!-- Boss Stats -->
                <div class="bg-black/75 border-2 border-pink-500 rounded-lg p-1.5 sm:p-2 w-44 sm:w-64 text-right">
                    <div class="flex justify-between items-center text-pink-300 mb-1 gap-1">
                        <span id="hud-stage-badge" class="bg-pink-600 text-white px-1 rounded text-[7px] sm:text-[8px]">STG 1/10</span>
                        <span id="hud-boss-name" class="truncate text-[8px] sm:text-[10px]">DINO MAID</span>
                    </div>
                    <div class="w-full h-2.5 bg-gray-900 border border-gray-600 rounded overflow-hidden">
                        <div id="hud-boss-hp-bar" class="h-full bg-gradient-to-r from-pink-500 via-red-500 to-yellow-500 transition-all duration-150" style="width: 100%;"></div>
                    </div>
                    <div id="hud-boss-hp-text" class="mt-1 text-[8px] sm:text-[9px] text-pink-200">
                        HP: 460/460
                    </div>
                </div>
            </div>
        </div> <!-- Home Screen Overlay -->
        <div id="start-screen" class="absolute inset-0 bg-gradient-to-b from-[#13092b]/95 via-[#1c0e3d]/90 to-[#090612]/95 flex flex-col items-center justify-center p-4 text-center z-20">
            <div class="text-[9px] sm:text-xs text-cyan-300 tracking-widest mb-1">1989 RETRO TRIVIA BATTLER</div>
            <h1 class="text-xl sm:text-4xl text-pink-500 neon-text-pink leading-snug mb-1">
                HERO VS DINO MAID
            </h1>
            <h2 class="text-sm sm:text-xl text-yellow-300 neon-text-gold mb-4">
                10-STAGE ARCADE CLASH
            </h2>

            <div class="bg-black/70 border-2 border-purple-500 rounded-xl p-3 sm:p-4 max-w-lg text-[8px] sm:text-[10px] leading-relaxed text-gray-200 mb-5 space-y-2">
                <p><span class="text-cyan-400">1. BATTLE:</span> Dodge &amp; blast the Tyrannosaurus Maid across <span class="text-pink-400">10 STAGES</span>! Clear a round for <span class="text-green-400">+10% MAX HP</span>!</p>
                <p><span class="text-yellow-300">2. KRILLION PAUSE (EVERY 15s):</span> Time freezes! Type the <span class="text-pink-400">RAREST</span> valid answer!</p>
                <p><span class="text-green-400">3. POWERS (&lt;10% RARITY):</span> 25% chance to unlock <span class="text-orange-400">1 PER-RUN POWER</span> (Flame, Lightning, Ice, or 1-in-50 Chrono Slow)!</p>
                <p class="text-gray-400 pt-1">CONTROLS: [A/D / ARROWS] Move &bull; [W/UP] Jump &bull; [SPACE/Z] Shoot &bull; [SHIFT/X] Dash</p>
            </div>  <button id="btn-start" class="pixel-btn animate-pulse-slow bg-pink-600 hover:bg-pink-500 text-white px-7 py-3.5 rounded-xl text-xs sm:text-base">
                CLASSIC
            </button>
        </div>

        <!-- Krillion Trivia Pause Modal -->
        <div id="quiz-overlay" class="hidden absolute inset-0 bg-black/85 backdrop-blur-sm flex flex-col items-center justify-center p-3 sm:p-6 z-30">
            <div class="bg-[#180d36] border-4 border-yellow-400 rounded-2xl p-4 sm:p-6 w-full max-w-xl shadow-[0_0_40px_rgba(255,215,0,0.45)] text-center">
                <div class="flex justify-between items-center text-[9px] sm:text-xs text-cyan-300 mb-2">
                    <span>TIME FROZEN!</span>
                    <span id="quiz-timer" class="text-yellow-300 text-sm sm:text-base">20.0s</span>
                </div>

                <div class="text-[10px] sm:text-xs text-pink-400 mb-1">NAME SOMETHING OBSCURE FOR:</div>
                <div id="quiz-prompt" class="text-sm sm:text-xl text-white neon-text-cyan bg-black/60 border-2 border-cyan-500 rounded-xl py-3 px-3 mb-3 leading-snug">
                    A TYPE OF FRUIT
                </div> <div id="quiz-input-area">
                    <input
                        id="quiz-input"
                        type="text"
                        autocomplete="off"
                        autocorrect="off"
                        spellcheck="false"
                        placeholder="TYPE ANY ANSWER..."
                        class="w-full bg-black border-3 border-pink-500 focus:border-yellow-300 outline-none rounded-xl px-3 py-3 text-center text-xs sm:text-base text-yellow-300 uppercase mb-3"
                    />
                    <button id="btn-submit-answer" class="pixel-btn bg-green-500 hover:bg-green-400 text-black px-6 py-2.5 rounded-xl text-xs sm:text-sm w-full">
                        LOCK IN ANSWER!
                    </button>
                </div>

                <!-- Result Reveal Box -->
                <div id="quiz-result-area" class="hidden space-y-2.5 mt-2">
                    <div id="quiz-autocorrect-note" class="text-[8px] sm:text-[10px] text-cyan-300"></div>
                    <div id="quiz-rarity-display" class="text-base sm:text-2xl text-yellow-300 neon-text-gold">
                        ONLY 4% OF PEOPLE SAID THAT!
                    </div> <!-- Rarity Percentage Bar -->
                    <div class="bg-black/80 border-2 border-purple-400 rounded-xl p-2.5">
                        <div class="flex justify-between text-[7px] sm:text-[9px] text-gray-300 mb-1">
                            <span>0% (ULTRA RARE)</span>
                            <span id="quiz-pct-label" class="text-yellow-300">4%</span>
                            <span>100% (COMMON)</span>
                        </div>
                        <div class="relative w-full h-4 bg-gray-900 border border-gray-600 rounded-lg overflow-hidden">
                            <div id="quiz-rarity-bar" class="h-full bg-gradient-to-r from-green-400 via-yellow-400 to-pink-500 transition-all duration-500 ease-out" style="width: 0%;"></div>
                            <!-- 10% Power Threshold Marker -->
                            <div class="absolute top-0 bottom-0 left-[10%] w-0.5 bg-cyan-300 shadow-[0_0_6px_#00f0ff]" title="10% Power Threshold"></div>
                        </div>
                        <div class="text-left text-[7px] text-cyan-300 mt-1" style="padding-left: 4%;">
                            &uarr; &lt;10% POWER CHANCE ZONE (25%)
                        </div>
                    </div>
                    <div id="quiz-reward-desc" class="text-[9px] sm:text-xs text-green-400 leading-relaxed bg-black/60 p-2.5 rounded-xl border border-green-500">
                        REWARD: TRIPLE PLASMA + HEAL 35 HP!    </div>
                    <button id="btn-resume-battle" class="pixel-btn bg-pink-600 hover:bg-pink-500 text-white px-6 py-2.5 rounded-xl text-xs sm:text-sm mt-2">
                        RESUME CLASH!
                    </button>
                </div>
            </div>
        </div>

        <!-- Game Over / Victory Screen -->
        <div id="end-screen" class="hidden absolute inset-0 bg-black/90 flex flex-col items-center justify-center p-4 text-center z-40">
            <h2 id="end-title" class="text-2xl sm:text-4xl text-pink-500 neon-text-pink mb-2">GAME OVER</h2>
            <p id="end-subtitle" class="text-[10px] sm:text-xs text-cyan-300 mb-4">THE DINO MAID SWEPT YOU AWAY!</p>

            <div class="bg-[#180d36] border-2 border-yellow-400 rounded-xl p-4 w-full max-w-md text-[9px] sm:text-xs space-y-2 mb-5">
                <div class="flex justify-between">
                    <span class="text-gray-400">STAGES CLEARED:</span>
                    <span id="end-stages" class="text-pink-400">0 / 10</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-400">TIME SURVIVED:</span>
                    <span id="end-time" class="text-white">0.0s</span>
                </div> <div class="flex justify-between">
                    <span class="text-gray-400">FINAL SCORE:</span>
                    <span id="end-score" class="text-green-400">0</span>
                </div>
            </div>

            <button id="btn-restart" class="pixel-btn bg-cyan-500 hover:bg-cyan-400 text-black px-6 py-3 rounded-xl text-xs sm:text-sm">
                PLAY AGAIN
            </button>
        </div>
    </div>

    <!-- Mobile On-Screen Arcade Controls -->
    <div id="mobile-controls" class="w-full max-w-3xl px-4 mt-3 justify-between items-center gap-4">
        <div class="flex gap-2">
            <button id="m-left" class="touch-pad-btn w-14 h-14">&larr;</button>
            <button id="m-right" class="touch-pad-btn w-14 h-14">&rarr;</button>
            <button id="m-jump" class="touch-pad-btn w-14 h-14">UP</button>
        </div>
        <div class="flex gap-2">
            <button id="m-dash" class="touch-pad-btn w-16 h-14 border-yellow-400 text-yellow-300 text-[10px]">DASH</button>
            <button id="m-fire" class="touch-pad-btn w-20 h-14 border-pink-500 bg-pink-700/80 text-white text-[10px]">FIRE</button>
        </div>
    </div> <script>
        // ============================================================================
        // 1. RETRO WEB AUDIO CHIPTUNE SYNTHESIZER (NO EXTERNAL AUDIO FILES)
        // ============================================================================
        class RetroSoundEngine {
            constructor() {
                this.ctx = null;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    if (AudioCtx) this.ctx = new AudioCtx();
                }
                if (this.ctx && this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            } playTone(freq, type, duration, vol = 0.12, slideTo = null) {
                if (!this.ctx) return;
                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = type;
                    osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
                    if (slideTo) {
                        osc.frequency.exponentialRampToValueAtTime(Math.max(20, slideTo), this.ctx.currentTime + duration);
                    }
                    gain.gain.setValueAtTime(vol, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.001, this.ctx.currentTime + duration);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start();
                    osc.stop(this.ctx.currentTime + duration);
                } catch (e) {}
            }

            shoot(powerLevel = 1) {
                const base = 520 + powerLevel * 90;
                this.playTone(base, 'square', 0.09, 0.08, 180);
            }

            jump() {
                this.playTone(220, 'square', 0.14, 0.09, 540);
            }dash() {
                this.playTone(300, 'sawtooth', 0.12, 0.08, 780);
            }

            bossHit() {
                this.playTone(140, 'sawtooth', 0.07, 0.09, 65);
            }

            playerHurt() {
                this.playTone(190, 'sawtooth', 0.25, 0.16, 45);
            }

            roar() {
                this.playTone(135, 'sawtooth', 0.45, 0.16, 42);
                setTimeout(() => this.playTone(95, 'square', 0.35, 0.12, 38), 110);
            }

            stageClear() {
                const notes = [392, 493.88, 587.33, 783.99, 987.77];
                notes.forEach((n, i) => {
                    setTimeout(() => this.playTone(n, 'square', 0.16, 0.11), i * 85);
                });
            } quizFreeze() {
                [880, 660, 440, 330].forEach((n, i) => {
                    setTimeout(() => this.playTone(n, 'triangle', 0.12, 0.11), i * 60);
                });
            }

            powerUp(rare = false) {
                const notes = rare
                    ? [523.25, 659.25, 783.99, 1046.5, 1318.5]
                    : [440, 554.37, 659.25];
                notes.forEach((n, i) => {
                    setTimeout(() => this.playTone(n, 'square', 0.14, 0.11), i * 75);
                });
            }
        }

        const sfx = new RetroSoundEngine();

        // ============================================================================
        // 2. KRILLION TRIVIA DATABASE & FUZZY AUTOCORRECT ENGINE
        // ============================================================================
        const TRIVIA_BANK = [
            {
                prompt: "A TYPE OF FRUIT",
                common: { "apple": 34, "banana": 26, "orange": 14, "strawberry": 8, "grape": 5, "grapes": 5,
                    "watermelon": 4, "mango": 3, "pineapple": 3, "blueberry": 2, "peach": 2, "pear": 2,
                    "kiwi": 2, "cherry": 2, "lemon": 1, "lime": 1, "raspberry": 1, "blackberry": 1,
                    "dragonfruit": 1, "papaya": 1, "pomegranate": 1, "lychee": 1, "durian": 1, "starfruit": 1,
                    "rambutan": 1, "jackfruit": 1, "persimmon": 1, "guava": 1, "fig": 1, "plum": 1, "apricot": 1
                },
                validSet: ["apple","banana","orange","strawberry","grape","grapes","watermelon","mango","pineapple","blueberry","peach","pear","kiwi","cherry","lemon","lime","raspberry","blackberry","dragonfruit","papaya","pomegranate","lychee","durian","starfruit","rambutan","jackfruit","persimmon","guava","fig","plum","apricot","cantaloupe","honeydew","cranberry","coconut","avocado","tomato","nectarine","tangerine","clementine","grapefruit","kumquat","quince","mulberry","boysenberry","elderberry","gooseberry","passionfruit","plantain","soursop","ackee","breadfruit","cherimoya","feijoa","jabuticaba","longan","loquat","mangosteen","miracle fruit","noni","pomelo","salak","tamarind","ugli fruit","yuzu"]
            },
            {
                prompt: "AN ANIMAL WITH FOUR LEGS",
                common: {
                    "dog": 38, "cat": 27, "horse": 9, "cow": 7, "lion": 5, "tiger": 4, "elephant": 3,
                    "bear": 3, "giraffe": 2, "zebra": 2, "deer": 2, "wolf": 2, "fox": 1, "pig": 1,
                    "sheep": 1, "goat": 1, "rabbit": 1, "squirrel": 1, "cheetah": 1, "hippo": 1, "rhino": 1
                }, validSet: ["dog","cat","horse","cow","lion","tiger","elephant","bear","giraffe","zebra","deer","wolf","fox","pig","sheep","goat","rabbit","squirrel","cheetah","hippo","rhino","leopard","jaguar","hyena","buffalo","bison","moose","elk","camel","llama","alpaca","donkey","mule","raccoon","skunk","badger","otter","beaver","hedgehog","porcupine","armadillo","anteater","sloth","tapir","okapi","capybara","wombat","koala","platypus","echidna","crocodile","alligator","turtle","tortoise","lizard","iguana","gecko","chameleon","salamander","newt","frog","toad","axolotl","mouse","rat","hamster","gerbil","guinea pig","ferret","weasel","mink","wolverine","lynx","bobcat","cougar","panther","panda","polar bear","grizzly bear"]
            },
            {
                prompt: "A COLOR",
                common: {
                    "blue": 35, "red": 28, "green": 15, "yellow": 8, "purple": 6, "black": 4, "pink": 4,
                    "orange": 3, "white": 3, "brown": 2, "gray": 1, "grey": 1, "violet": 1, "indigo": 1,
                    "turquoise": 1, "magenta": 1, "cyan": 1, "teal": 1, "maroon": 1, "chartreuse": 1
                },
                validSet: ["blue","red","green","yellow","purple","black","pink","orange","white","brown","gray","grey","violet","indigo","turquoise","magenta","cyan","teal","maroon","chartreuse","beige","tan","ivory","cream","gold","silver","bronze","copper","crimson","scarlet","burgundy","coral","salmon","peach","amber","ochre","mustard","olive","lime","mint","emerald","jade","sage","forest green","navy","cobalt","sapphire","azure","cerulean","sky blue","lavender","lilac","plum","mauve","periwinkle","fuchsia","rose","vermilion","sienna","umber","taupe","charcoal","ebony","alabaster","ecru","sepia","viridian","heliotrope","amaranth","celadon","gamboge","phthalo"]
            },
            {  prompt: "SOMETHING IN A KITCHEN",
                common: {
                    "knife": 22, "fork": 18, "spoon": 18, "refrigerator": 14, "fridge": 14, "stove": 12,
                    "oven": 11, "microwave": 10, "plate": 8, "bowl": 7, "pan": 6, "pot": 6, "sink": 5,
                    "toaster": 4, "blender": 3, "cutting board": 2, "spatula": 2, "whisk": 1, "colander": 1
                },
                validSet: ["knife","fork","spoon","refrigerator","fridge","stove","oven","microwave","plate","bowl","pan","pot","sink","toaster","blender","cutting board","spatula","whisk","colander","ladle","tongs","peeler","grater","strainer","rolling pin","measuring cup","kettle","teapot","coffee maker","dishwasher","freezer","cabinet","drawer","counter","island","faucet","sponge","dish soap","paper towels","napkin","cup","mug","glass","wine glass","pitcher","baking sheet","muffin tin","casserole dish","skillet","wok","dutch oven","slow cooker","air fryer","mixer","food processor","can opener","corkscrew","bottle opener","ice tray","trash can","spice rack","salt shaker","pepper mill","mortar and pestle","zester","mandoline","garlic press","baster","thermometer","trivet","oven mitt","apron"]
            },
            {
                prompt: "A COUNTRY IN EUROPE",
                common: {"france": 28, "italy": 22, "germany": 16, "spain": 14, "england": 10, "uk": 10,
                    "united kingdom": 10, "greece": 6, "sweden": 4, "norway": 3, "poland": 3, "ireland": 3,
                    "portugal": 2, "switzerland": 2, "netherlands": 2, "belgium": 2, "austria": 2, "denmark": 1
                },
                validSet: ["france","italy","germany","spain","england","uk","united kingdom","greece","sweden","norway","poland","ireland","portugal","switzerland","netherlands","belgium","austria","denmark","finland","iceland","scotland","wales","czech republic","czechia","slovakia","hungary","romania","bulgaria","croatia","serbia","slovenia","bosnia","montenegro","albania","north macedonia","ukraine","belarus","lithuania","latvia","estonia","moldova","luxembourg","monaco","andorra","liechtenstein","san marino","vatican city","malta","cyprus"]
            },
            { prompt: "A SPORT PLAYED WITH A BALL",
                common: {
                    "soccer": 32, "football": 28, "basketball": 24, "baseball": 14, "tennis": 9, "golf": 7,
                    "volleyball": 6, "rugby": 3, "cricket": 3, "bowling": 2, "ping pong": 2, "table tennis": 2,
                    "dodgeball": 1, "kickball": 1, "handball": 1, "polo": 1, "water polo": 1, "lacrosse": 1
                },
                validSet: ["soccer","football","basketball","baseball","tennis","golf","volleyball","rugby","cricket","bowling","ping pong","table tennis","dodgeball","kickball","handball","polo","water polo","lacrosse","softball","squash","racquetball","pickleball","bocce","croquet","pool","billiards","snooker","tetherball","four square","netball","australian rules football","gaelic football","hurling","sepak takraw","jai alai","padel","beach volleyball","field hockey"]
            },
            { prompt: "A MUSICAL INSTRUMENT",
                common: {
                    "guitar": 34, "piano": 28, "drums": 16, "violin": 10, "trumpet": 6, "flute": 5,
                    "saxophone": 4, "bass": 3, "cello": 2, "clarinet": 2, "trombone": 2, "harp": 2,
                    "ukulele": 2, "banjo": 1, "harmonica": 1, "accordion": 1, "oboe": 1, "tuba": 1
                },
                validSet: ["guitar","piano","drums","violin","trumpet","flute","saxophone","bass","cello","clarinet","trombone","harp","ukulele","banjo","harmonica","accordion","oboe","tuba","viola","double bass","french horn","bassoon","piccolo","xylophone","marimba","vibraphone","timpani","cymbals","tambourine","maracas","triangle","bongos","congas","djembe","organ","harpsichord","synthesizer","keytar","mandolin","sitar","lute","bagpipes","kazoo","recorder","ocarina","pan flute","didgeridoo","kalimba","theremin","balalaika","dulcimer","zither","lyre","cornet","flugelhorn","euphonium","sousaphone","gong","chimes","castanets","cowbell"]
            },
            {
                prompt: "SOMETHING YOU WEAR ON YOUR FEET",
                common: {
                    "shoes": 38, "socks": 28, "sneakers": 14, "boots": 10, "sandals": 7, "flip flops": 5,
                    "slippers": 4, "heels": 3, "high heels": 3, "crocs": 2, "cleats": 1, "loafers": 1
                },
                validSet: ["shoes","socks","sneakers","boots","sandals","flip flops","slippers","heels","high heels","crocs","cleats","loafers","moccasins","clogs","espadrilles","oxfords","brogues","ballet flats","pumps","stilettos","wedges","platforms","slides","mules","galoshes","rain boots","snow boots","hiking boots","cowboy boots","combat boots","steel toe boots","skates","ice skates","roller skates","rollerblades","ski boots","snowshoes","flippers","fins","pointe shoes","tap shoes","bowling shoes","golf shoes","stockings","tights","leg warmers","footies"]     },
            {
                prompt: "A PLANET OR CELESTIAL BODY IN OUR SOLAR SYSTEM",
                common: {
                    "earth": 26, "mars": 24, "jupiter": 16, "saturn": 14, "venus": 9, "mercury": 7,
                    "uranus": 6, "neptune": 6, "pluto": 5, "sun": 4, "moon": 4, "europa": 1, "titan": 1
                },
                validSet: ["earth","mars","jupiter","saturn","venus","mercury","uranus","neptune","pluto","sun","moon","europa","titan","ganymede","callisto","io","enceladus","triton","phobos","deimos","charon","ceres","eris","makemake","haumea","sedna","halley's comet","asteroid","comet","meteor","oort cloud","kuiper belt","vesta","pallas","hygiea","miranda","ariel","umbriel","titania","oberon","iapetus","rhea","dione","tethys","mimas"]
            },
            {
                prompt: "A VEHICLE OR MODE OF TRANSPORT",
                common: {
                    "car": 40, "truck": 16, "bus": 12, "train": 10, "airplane": 9, "plane": 9, "bicycle": 7,
                    "bike": 7, "motorcycle": 6, "boat": 4, "helicopter": 3, "subway": 2, "van": 2, "scooter": 2
                }, validSet: ["car","truck","bus","train","airplane","plane","bicycle","bike","motorcycle","boat","helicopter","subway","van","scooter","skateboard","rollerblades","ship","ferry","yacht","sailboat","canoe","kayak","raft","submarine","hovercraft","jet ski","snowmobile","tractor","bulldozer","excavator","forklift","ambulance","fire truck","police car","taxi","cab","limousine","rv","camper","trailer","tram","trolley","monorail","cable car","gondola","hot air balloon","blimp","zeppelin","glider","rocket","spaceship","rickshaw","chariot","carriage","wagon","sled","sleigh","unicycle","tricycle","moped","segway","atv","quad"]
            },
            {
                prompt: "A PIZZA TOPPING",
                common: {
                    "pepperoni": 42, "cheese": 22, "mushroom": 12, "mushrooms": 12, "sausage": 10,
                    "pineapple": 7, "olives": 5, "peppers": 4, "onions": 4, "bacon": 4, "ham": 3,
                    "spinach": 2, "jalapenos": 2, "anchovies": 1, "chicken": 1, "garlic": 1, "basil": 1
                },
                validSet: ["pepperoni","cheese","mushroom","mushrooms","sausage","pineapple","olives","black olives","green olives","peppers","bell peppers","green peppers","onions","red onions","bacon","ham","spinach","jalapenos","anchovies","chicken","bbq chicken","garlic","basil","tomatoes","sun dried tomatoes","artichoke","artichoke hearts","broccoli","feta","ricotta","parmesan","mozzarella","gorgonzola","provolone","meatballs","salami","prosciutto","chorizo","capers","corn","egg","truffle","pesto","arugula","banana peppers","roasted garlic"]
            },
            {prompt: "A SCHOOL SUBJECT",
                common: {
                    "math": 38, "mathematics": 38, "science": 24, "english": 18, "history": 15,
                    "art": 6, "music": 4, "pe": 4, "gym": 4, "biology": 3, "chemistry": 3, "physics": 2,
                    "geography": 2, "spanish": 2, "french": 1, "computer science": 1, "psychology": 1
                },
                validSet: ["math","mathematics","science","english","history","art","music","pe","physical education","gym","biology","chemistry","physics","geography","spanish","french","german","latin","mandarin","computer science","coding","programming","psychology","sociology","economics","government","civics","philosophy","literature","reading","writing","drama","theater","dance","band","choir","orchestra","health","home economics","woodshop","metalshop","auto shop","photography","astronomy","geology","calculus","algebra","geometry","trigonometry","statistics","anatomy","environmental science"]
            },
            {
                prompt: "A SEA CREATURE",
                common: {
                    "shark": 28, "whale": 20, "dolphin": 18, "octopus": 12, "fish": 10, "crab": 6,
                    "lobster": 5, "jellyfish": 5, "starfish": 4, "turtle": 4, "sea turtle": 4, "squid": 3,
                    "stingray": 2, "seahorse": 2, "seal": 2, "walrus": 1, "eel": 1, "clam": 1, "shrimp": 1
                },validSet: ["shark","whale","dolphin","octopus","fish","crab","lobster","jellyfish","starfish","turtle","sea turtle","squid","stingray","seahorse","seal","walrus","eel","moray eel","clam","shrimp","prawn","oyster","scallop","mussel","urchin","sea urchin","sea cucumber","anemone","coral","sponge","manatee","dugong","narwhal","beluga","orca","killer whale","manta ray","hammerhead","great white","swordfish","marlin","tuna","salmon","clownfish","pufferfish","anglerfish","barracuda","herring","sardine","anchovy","cod","halibut","flounder","krill","plankton","barnacle","nautilus","cuttlefish","horseshoe crab","sea lion","sea otter","penguin"]
            },
            {
                prompt: "A WEATHER CONDITION",
                common: {
                    "rain": 35, "snow": 24, "sunny": 16, "sun": 16, "wind": 10, "windy": 10, "cloudy": 8,
                    "thunderstorm": 6, "storm": 6, "tornado": 4, "hurricane": 4, "fog": 3, "foggy": 3,
                    "hail": 3, "sleet": 2, "lightning": 2, "blizzard": 2, "drizzle": 1, "humidity": 1
                },
                validSet: ["rain","snow","sunny","sun","wind","windy","cloudy","overcast","thunderstorm","storm","tornado","hurricane","typhoon","cyclone","fog","foggy","mist","haze","smog","hail","sleet","freezing rain","lightning","thunder","blizzard","drizzle","downpour","monsoon","drought","heatwave","cold front","frost","ice","black ice","flurries","squall","gale","breeze","dust storm","sandstorm","rainbow","humidity","humid","clear"]
            },
            { prompt: "A BODY PART",
                common: {
                    "hand": 24, "arm": 20, "leg": 18, "foot": 15, "head": 14, "eye": 10, "eyes": 10,
                    "nose": 8, "mouth": 7, "ear": 6, "ears": 6, "finger": 5, "toe": 4, "heart": 3,
                    "brain": 3, "stomach": 2, "knee": 2, "elbow": 2, "shoulder": 2, "neck": 1, "back": 1
                },
                validSet: ["hand","arm","leg","foot","feet","head","eye","eyes","nose","mouth","ear","ears","finger","fingers","thumb","toe","toes","heart","brain","stomach","knee","elbow","shoulder","neck","back","chest","rib","ribs","spine","skull","hair","face","forehead","eyebrow","eyelash","cheek","chin","jaw","lip","lips","tooth","teeth","tongue","throat","wrist","ankle","heel","shin","calf","thigh","hip","waist","belly","navel","lung","lungs","liver","kidney","spleen","pancreas","intestine","appendix","collarbone","clavicle","scapula","femur","patella","knuckle","palm","sole","arch","nostril","earlobe","pupil","iris","retina","cornea"]
            },
            {
                prompt: "A BOARD GAME OR CARD GAME",
                common: {
                    "monopoly": 36, "chess": 22, "checkers": 15, "uno": 12, "scrabble": 9, "clue": 7,
                    "risk": 6, "poker": 5, "battleship": 4, "life": 4, "candy land": 3, "sorry": 3,
                    "catan": 3, "settlers of catan": 3, "connect four": 2, "jenga": 2, "blackjack": 2, "solitaire": 2
                }, validSet: ["monopoly","chess","checkers","uno","scrabble","clue","risk","poker","battleship","life","game of life","candy land","sorry","catan","settlers of catan","connect four","jenga","blackjack","solitaire","go fish","war","crazy eights","hearts","spades","bridge","rummy","gin rummy","cribbage","pinochle","euchre","baccarat","old maid","snap","slapjack","phase 10","skip bo","dos","exploding kittens","cards against humanity","apples to apples","pictionary","trivial pursuit","yahtzee","backgammon","go","othello","reversi","mancala","chutes and ladders","snakes and ladders","operation","mousetrap","hungry hungry hippos","guess who","stratego","ticket to ride","carcassonne","pandemic","wingspan","azul","dungeons and dragons","magic the gathering"]
            },
            {
                prompt: "A TYPE OF DESSERT OR SWEET TREAT",
                common: {
                    "ice cream": 34, "cake": 26, "cookie": 16, "cookies": 16, "brownie": 12, "brownies": 12,
                    "pie": 9, "donut": 7, "doughnut": 7, "cupcake": 6, "chocolate": 5, "pudding": 4,
                    "cheesecake": 3, "macaron": 1, "tiramisu": 1, "gelato": 1, "baklava": 1, "churro": 1
                },
                validSet: ["ice cream","cake","cookie","cookies","brownie","brownies","pie","donut","doughnut","cupcake","chocolate","pudding","cheesecake","macaron","tiramisu","gelato","baklava","churro","eclair","cannoli","creme brulee","flan","mousse","fudge","truffle","sorbet","sundae","milkshake","popsicle","cobbler","crumble","tart","pastry","strudel","profiterole","meringue","pavlova","parfait","custard","jelly","jello","marshmallow","candy","caramel","toffee","nougat","beignet","mochi","biscotti","shortbread","snickerdoodle","gingerbread","red velvet"] },
            {
                prompt: "A SUPERHERO",
                common: {
                    "batman": 32, "superman": 28, "spider-man": 26, "spiderman": 26, "iron man": 14,
                    "wonder woman": 11, "captain america": 9, "hulk": 7, "thor": 6, "flash": 5,
                    "wolverine": 3, "black panther": 3, "aquaman": 2, "deadpool": 2, "green lantern": 1
                },
                validSet: ["batman","superman","spider-man","spiderman","iron man","wonder woman","captain america","hulk","thor","flash","the flash","wolverine","black panther","aquaman","deadpool","green lantern","black widow","hawkeye","doctor strange","scarlet witch","vision","ant-man","wasp","captain marvel","star-lord","groot","rocket raccoon","gamora","drax","daredevil","punisher","luke cage","iron fist","jessica jones","cyclops","storm","jean grey","rogue","gambit","beast","nightcrawler","professor x","magneto","silver surfer","human torch","invisible woman","mister fantastic","thing","supergirl","batgirl","robin","nightwing","cyborg","starfire","raven","beast boy","shazam","martian manhunter","green arrow","black canary","zatanna","constantine","spawn","hellboy","invincible","omnidman","homelander"]
            },
            { prompt: "A TYPE OF DINOSAUR",
                common: {
                    "t-rex": 42, "tyrannosaurus": 42, "tyrannosaurus rex": 42, "triceratops": 22,
                    "stegosaurus": 14, "velociraptor": 12, "raptor": 12, "brachiosaurus": 8,
                    "pterodactyl": 6, "brontosaurus": 5, "ankylosaurus": 3, "spinosaurus": 3, "diplodocus": 2
                },
                validSet: ["t-rex","tyrannosaurus","tyrannosaurus rex","triceratops","stegosaurus","velociraptor","raptor","brachiosaurus","pterodactyl","pteranodon","brontosaurus","ankylosaurus","spinosaurus","diplodocus","allosaurus","parasaurolophus","iguanodon","pachycephalosaurus","carnotaurus","giganotosaurus","gallimimus","deinonychus","compsognathus","dilophosaurus","ceratosaurus","corythosaurus","edmontosaurus","maiasaura","oviraptor","therizinosaurus","utahraptor","baryonyx","albertosaurus","daspletosaurus","tarbosaurus","styracosaurus","centrosaurus","protoceratops","euoplocephalus","kentrosaurus","apatosaurus","camarasaurus","argentinosaurus","patagotitan","dreadnoughtus","archaeopteryx","microraptor","plesiosaurus","mosasaurus","ichthyosaurus"]
            },
            { prompt: "A VEGETABLE",
                common: {
                    "carrot": 32, "broccoli": 26, "potato": 18, "corn": 14, "lettuce": 10, "spinach": 8,
                    "onion": 7, "tomato": 6, "celery": 5, "cucumber": 5, "peas": 4, "green beans": 3,
                    "cauliflower": 3, "asparagus": 2, "zucchini": 2, "eggplant": 1, "radish": 1, "kale": 1
                },
                validSet: ["carrot","broccoli","potato","corn","lettuce","spinach","onion","tomato","celery","cucumber","peas","green beans","cauliflower","asparagus","zucchini","eggplant","radish","kale","cabbage","brussels sprouts","artichoke","beet","turnip","parsnip","yam","sweet potato","bell pepper","jalapeno","garlic","ginger","leek","shallot","scallion","bok choy","chard","swiss chard","collard greens","mustard greens","arugula","watercress","endive","radicchio","fennel","kohlrabi","rutabaga","jicama","okra","squash","butternut squash","acorn squash","pumpkin","rhubarb","bamboo shoots","edamame","chickpea","lentil"]
            },
            {
                prompt: "A COLD OR HOT DRINK",
                common: {
                    "water": 38, "coffee": 26, "tea": 18, "soda": 14, "milk": 12, "orange juice": 9,
                    "juice": 9, "lemonade": 7, "hot chocolate": 5, "beer": 4, "wine": 3, "smoothie": 2,
                    "apple juice": 2, "milkshake": 2, "espresso": 1, "latte": 1, "matcha": 1, "kombucha": 1
                },
                validSet: ["water","coffee","tea","soda","pop","cola","coke","pepsi","sprite","root beer","ginger ale","milk","orange juice","juice","apple juice","grape juice","cranberry juice","lemonade","limeade","iced tea","sweet tea","green tea","black tea","herbal tea","chamomile","hot chocolate","hot cocoa","beer","wine","champagne","cider","apple cider","smoothie","milkshake","espresso","latte","cappuccino","americano","mocha","macchiato","matcha","chai","kombucha","seltzer","sparkling water","club soda","tonic water","energy drink","sports drink","gatorade","horchata","boba","bubble tea","eggnog","punch"]
            },
            { prompt: "AN ITEM OF CLOTHING",
                common: {
                    "shirt": 34, "pants": 26, "jeans": 16, "dress": 12, "jacket": 10, "coat": 9,
                    "sweater": 8, "hoodie": 7, "shorts": 7, "skirt": 6, "hat": 5, "socks": 4,
                    "underwear": 3, "suit": 2, "tie": 2, "scarf": 2, "gloves": 1, "vest": 1
                },
                validSet: ["shirt","t-shirt","pants","jeans","dress","jacket","coat","sweater","hoodie","sweatshirt","shorts","skirt","hat","cap","beanie","socks","underwear","boxers","briefs","bra","suit","tuxedo","tie","bowtie","scarf","gloves","mittens","vest","cardigan","blouse","polo","tank top","camisole","leggings","tights","sweatpants","joggers","slacks","trousers","khakis","chinos","overalls","jumpsuit","romper","pajamas","bathrobe","robe","swimsuit","bikini","trunks","parka","trench coat","blazer","poncho","shawl","kimono","apron","uniform"]
            },
            {
                prompt: "A JOB OR PROFESSION",
                common: {
                    "doctor": 32, "teacher": 26, "police officer": 16, "cop": 16, "firefighter": 14,
                    "nurse": 12, "lawyer": 10, "engineer": 8, "chef": 6, "cook": 6, "pilot": 5,
                    "dentist": 4, "scientist": 4, "artist": 3, "actor": 3, "astronaut": 2, "mechanic": 2
                },validSet: ["doctor","teacher","police officer","cop","firefighter","nurse","lawyer","attorney","engineer","chef","cook","pilot","dentist","scientist","artist","actor","actress","astronaut","mechanic","plumber","electrician","carpenter","architect","accountant","banker","cashier","waiter","waitress","bartender","barista","baker","butcher","farmer","fisherman","veterinarian","vet","surgeon","pharmacist","paramedic","emt","judge","politician","mayor","president","soldier","sailor","marine","journalist","reporter","writer","author","editor","photographer","musician","singer","dancer","athlete","coach","referee","librarian","professor","principal","janitor","maid","butler","gardener","florist","barber","hairdresser","tailor","blacksmith","welder","truck driver","bus driver"]
            },
            {
                prompt: "A MYTHICAL CREATURE",
                common: {
                    "dragon": 38, "unicorn": 26, "phoenix": 12, "mermaid": 11, "vampire": 9, "werewolf": 8,
                    "griffin": 5, "centaur": 5, "pegasus": 4, "fairy": 4, "elf": 3, "goblin": 3,
                    "troll": 2, "giant": 2, "minotaur": 2, "kraken": 1, "cyclops": 1, "hydra": 1
                },
                validSet: ["dragon","unicorn","phoenix","mermaid","merman","vampire","werewolf","griffin","gryphon","centaur","pegasus","fairy","pixie","elf","goblin","troll","giant","ogre","minotaur","kraken","cyclops","hydra","cerberus","chimera","sphinx","medusa","gorgon","siren","harpy","banshee","ghost","zombie","mummy","skeleton","lich","demon","angel","imp","leprechaun","gnome","dwarf","hobbit","orc","sasquatch","bigfoot","yeti","loch ness monster","chupacabra","wendigo","basilisk","cockatrice","manticore","wyvern","leviathan","behemoth","kitsune","tanuki","kappa","oni","djinn","genie","valkyrie","nymph","satyr","faun","kelpie","selkie"]
            },
            { prompt: "A HOLIDAY OR CELEBRATION",
                common: {
                    "christmas": 42, "halloween": 24, "thanksgiving": 18, "easter": 12, "new year": 9,
                    "new years": 9, "valentine's day": 8, "valentines day": 8, "fourth of july": 7,
                    "independence day": 7, "hanukkah": 4, "st patrick's day": 3, "labor day": 2, "memorial day": 2
                },
                validSet: ["christmas","halloween","thanksgiving","easter","new year","new years","new year's day","new year's eve","valentine's day","valentines day","fourth of july","4th of july","independence day","hanukkah","chanukah","st patrick's day","saint patrick's day","labor day","memorial day","veterans day","mother's day","father's day","birthday","anniversary","wedding","graduation","kwanzaa","diwali","eid","eid al-fitr","eid al-adha","ramadan","passover","rosh hashanah","yom kippur","chinese new year","lunar new year","cinco de mayo","dia de los muertos","day of the dead","mardi gras","carnival","oktoberfest","april fools","april fools day","earth day","arbor day","flag day","columbus day","indigenous peoples day","juneteenth","boxing day","groundhog day","presidents day","bastille day"]
            },
            {prompt: "SOMETHING FOUND IN A BEDROOM",
                common: {
                    "bed": 44, "pillow": 24, "blanket": 16, "lamp": 12, "dresser": 10, "closet": 9,
                    "mattress": 7, "sheets": 6, "alarm clock": 5, "nightstand": 5, "mirror": 4, "desk": 3,
                    "rug": 2, "curtains": 2, "wardrobe": 1, "bookshelf": 1, "hamper": 1, "comforter": 1
                },
                validSet: ["bed","pillow","blanket","lamp","dresser","closet","mattress","sheets","bedsheets","alarm clock","clock","nightstand","bedside table","mirror","desk","chair","rug","carpet","curtains","blinds","drapes","wardrobe","armoire","bookshelf","bookcase","hamper","laundry basket","comforter","duvet","quilt","pillowcase","headboard","footboard","bed frame","box spring","fan","ceiling fan","tv","television","poster","painting","clothes","hangers","slippers","stuffed animal","teddy bear","books","phone charger","outlet","window","door"]
            },
            {
                prompt: "A TYPE OF BIRD",
                common: {
                    "eagle": 28, "robin": 18, "crow": 14, "owl": 14, "pigeon": 12, "hawk": 10,
                    "sparrow": 8, "penguin": 8, "flamingo": 6, "parrot": 6, "duck": 5, "chicken": 5,
                    "seagull": 4, "blue jay": 4, "cardinal": 3, "hummingbird": 3, "woodpecker": 2, "ostrich": 2
                }, validSet: ["eagle","bald eagle","robin","crow","raven","owl","pigeon","dove","hawk","falcon","sparrow","penguin","flamingo","parrot","macaw","cockatoo","parakeet","budgie","duck","mallard","goose","swan","chicken","rooster","hen","turkey","seagull","gull","pelican","albatross","puffin","blue jay","cardinal","hummingbird","woodpecker","ostrich","emu","kiwi","cassowary","peacock","peafowl","pheasant","quail","partridge","stork","heron","crane","egret","ibis","toucan","kingfisher","magpie","starling","finch","goldfinch","canary","swallow","swift","wren","thrush","mockingbird","oriole","warbler","vulture","condor","osprey","kestrel"]
            },
            {
                prompt: "A BREAKFAST FOOD",
                common: {
                    "eggs": 36, "egg": 36, "pancakes": 26, "bacon": 22, "waffles": 16, "cereal": 14,
                    "toast": 12, "oatmeal": 7, "sausage": 6, "bagel": 5, "muffin": 4, "french toast": 4,
                    "hash browns": 3, "yogurt": 3, "croissant": 2, "omelet": 2, "omelette": 2, "granola": 1
                },
                validSet: ["eggs","egg","scrambled eggs","fried eggs","poached eggs","pancakes","bacon","waffles","cereal","toast","oatmeal","porridge","sausage","bagel","muffin","french toast","hash browns","yogurt","croissant","omelet","omelette","granola","donut","doughnut","cinnamon roll","danish","scone","biscuit","gravy","biscuits and gravy","grits","crepe","crepes","breakfast burrito","breakfast sandwich","english muffin","pop tart","fruit","banana","strawberry","blueberry","grapefruit","smoothie","acai bowl","quiche","frittata","benedict","eggs benedict","lox","smoked salmon","kippers","beans","baked beans"]
            },
            {
                prompt: "A TOOL IN A TOOLBOX",  common: {
                    "hammer": 42, "screwdriver": 28, "wrench": 18, "pliers": 12, "saw": 10, "drill": 8,
                    "tape measure": 7, "level": 4, "socket wrench": 3, "allen wrench": 3, "hex key": 3,
                    "utility knife": 2, "chisel": 2, "clamp": 1, "mallet": 1, "crowbar": 1, "file": 1
                },
                validSet: ["hammer","screwdriver","phillips screwdriver","flathead screwdriver","wrench","adjustable wrench","socket wrench","allen wrench","hex key","pliers","needle nose pliers","channel locks","vice grips","saw","handsaw","hacksaw","drill","power drill","cordless drill","tape measure","measuring tape","level","spirit level","utility knife","box cutter","chisel","clamp","c-clamp","bar clamp","mallet","rubber mallet","sledgehammer","crowbar","pry bar","file","rasp","sandpaper","stud finder","multimeter","voltage tester","wire strippers","crimper","soldering iron","flashlight","nails","screws","bolts","nuts","washers","duct tape","electrical tape","wood glue"]
            },
            {
                prompt: "A TYPE OF DOG BREED",
                common: {
                    "golden retriever": 30, "labrador": 26, "lab": 26, "german shepherd": 20, "poodle": 14,
                    "bulldog": 12, "beagle": 10, "chihuahua": 8, "husky": 8, "pug": 7, "dachshund": 5,
                    "rottweiler": 4, "boxer": 4, "corgi": 4, "dalmatian": 3, "pitbull": 3, "shih tzu": 2
                }, validSet: ["golden retriever","labrador","labrador retriever","lab","german shepherd","poodle","bulldog","french bulldog","english bulldog","beagle","chihuahua","husky","siberian husky","alaskan malamute","pug","dachshund","wiener dog","rottweiler","boxer","corgi","pembroke welsh corgi","dalmatian","pitbull","american pit bull terrier","shih tzu","yorkie","yorkshire terrier","border collie","collie","australian shepherd","bernese mountain dog","great dane","mastiff","saint bernard","newfoundland","doberman","doberman pinscher","greyhound","whippet","basset hound","bloodhound","cocker spaniel","springer spaniel","maltese","pomeranian","papillon","bichon frise","schnauzer","miniature schnauzer","jack russell terrier","boston terrier","scottish terrier","west highland white terrier","akita","shiba inu","chow chow","samoyed","weimaraner","vizsla","rhodesian ridgeback"]
            },
            {
                prompt: "SOMETHING AT A BEACH",
                common: {
                    "sand": 40, "water": 28, "ocean": 24, "waves": 16, "seashells": 14, "shells": 14,
                    "towel": 12, "umbrella": 10, "sunscreen": 9, "crab": 7, "seagull": 6, "sandcastle": 5,
                    "surfboard": 4, "palm tree": 3, "starfish": 3, "driftwood": 2, "kelp": 1, "seaweed": 2
                },
                validSet: ["sand","water","ocean","sea","waves","seashells","shells","towel","beach towel","umbrella","beach umbrella","sunscreen","sunblock","crab","hermit crab","seagull","sandcastle","surfboard","boogie board","bodyboard","paddleboard","palm tree","starfish","driftwood","kelp","seaweed","sunglasses","flip flops","sandals","swimsuit","bathing suit","trunks","bikini","beach ball","frisbee","cooler","ice chest","beach chair","lounge chair","lifeguard","lifeguard tower","pier","boardwalk","dock","boat","sailboat","jet ski","dolphin","jellyfish","clam","sand dollar","pebbles","rocks","tide pool","sun","sunlight","breeze"]
            },
            {prompt: "A VIDEO GAME CHARACTER",
                common: {
                    "mario": 42, "luigi": 22, "sonic": 20, "pikachu": 18, "link": 14, "zelda": 12,
                    "master chief": 8, "pac-man": 8, "pacman": 8, "donkey kong": 7, "kirby": 6, "bowser": 6,
                    "peach": 5, "princess peach": 5, "kratos": 4, "lara croft": 3, "cloud": 2, "samus": 2
                },
                validSet: ["mario","luigi","sonic","sonic the hedgehog","pikachu","link","zelda","princess zelda","master chief","pac-man","pacman","donkey kong","diddy kong","kirby","bowser","peach","princess peach","yoshi","toad","wario","waluigi","kratos","lara croft","cloud","cloud strife","sephiroth","tifa","aerith","samus","samus aran","ridley","fox mccloud","falco","captain falcon","ness","lucas","marth","roy","ike","chrom","lucina","byleth","shulk","villager","isabelle","tom nook","steve","alex","creeper","enderman","ryu","ken","chun-li","guile","scorpion","sub-zero","raiden","johnny cage","crash bandicoot","spyro","ratchet","clank","jak","daxter","sly cooper","nathan drake","joel","ellie","arthur morgan","john marston","geralt","ciri","doomguy","doom slayer","gordon freeman","chell","glados","tracer","genji","dva","mercy","sans","papyrus","frisk","cuphead","mugman","hollow knight","hornet","shovel knight","mega man","zero","protoman","solid snake","big boss","raiden","dante","vergil","bayonetta","joker","leon kennedy","jill valentine","chris redfield","albert wesker"]
            },
            {  prompt: "SOMETHING IN SPACE",
                common: {
                    "star": 32, "stars": 32, "planet": 26, "planets": 26, "moon": 22, "sun": 18,
                    "asteroid": 12, "comet": 10, "black hole": 9, "galaxy": 8, "meteor": 7, "rocket": 5,
                    "spaceship": 5, "astronaut": 4, "satellite": 4, "nebula": 3, "supernova": 2, "quasar": 1
                },
                validSet: ["star","stars","planet","planets","moon","sun","asteroid","comet","black hole","white dwarf","red giant","neutron star","pulsar","quasar","galaxy","milky way","andromeda","meteor","meteorite","meteoroid","shooting star","rocket","spaceship","spacecraft","astronaut","cosmonaut","satellite","space station","iss","telescope","hubble","james webb","nebula","supernova","constellation","orbit","gravity","vacuum","dark matter","dark energy","exoplanet","dwarf planet","pluto","mars","jupiter","saturn","venus","mercury","uranus","neptune","earth","kuiper belt","oort cloud","asteroid belt","solar flare","eclipse","aurora","cosmic dust","space debris"]
            },
            {
                prompt: "A TYPE OF FLOWER",
                common: {
                    "rose": 42, "tulip": 22, "sunflower": 18, "daisy": 16, "lily": 10, "orchid": 8,
                    "dandelion": 7, "daffodil": 6, "carnation": 5, "violet": 4, "lavender": 4, "poppy": 3,
                    "marigold": 3, "hibiscus": 2, "peony": 2, "hydrangea": 2, "iris": 1, "chrysanthemum": 1
                }, validSet: ["rose","tulip","sunflower","daisy","lily","water lily","tiger lily","calla lily","orchid","dandelion","daffodil","carnation","violet","pansy","lavender","poppy","marigold","hibiscus","peony","hydrangea","iris","chrysanthemum","mum","geranium","petunia","begonia","azalea","rhododendron","camellia","magnolia","jasmine","gardenia","honeysuckle","lilac","wisteria","cherry blossom","sakura","lotus","buttercup","bluebell","foxglove","snapdragon","zinnia","cosmos","aster","dahlia","gladiolus","freesia","anemone","ranunculus","delphinium","lupine","heather","thistle","clover","edelweiss","bird of paradise","plumeria","frangipani"]
            },
            {
                prompt: "A PIECE OF FURNITURE",
                common: {
                    "chair": 36, "table": 30, "couch": 24, "sofa": 24, "bed": 20, "desk": 12,
                    "dresser": 9, "bookshelf": 7, "coffee table": 6, "nightstand": 5, "recliner": 4,
                    "stool": 4, "bench": 3, "cabinet": 3, "wardrobe": 2, "ottoman": 2, "futon": 1, "loveseat": 1
                },
                validSet: ["chair","armchair","rocking chair","dining chair","office chair","table","dining table","coffee table","end table","side table","couch","sofa","sectional","loveseat","bed","bunk bed","crib","desk","computer desk","dresser","chest of drawers","bookshelf","bookcase","nightstand","bedside table","recliner","stool","bar stool","footstool","bench","cabinet","china cabinet","filing cabinet","wardrobe","armoire","ottoman","pouf","futon","daybed","chaise lounge","bean bag","vanity","hutch","sideboard","buffet","credenza","console table","tv stand","entertainment center","coat rack","hat rack","shoe rack","hammock"]},
            {
                prompt: "A SHAPE",
                common: {
                    "circle": 38, "square": 32, "triangle": 26, "rectangle": 16, "oval": 9, "star": 8,
                    "diamond": 7, "heart": 6, "hexagon": 5, "octagon": 5, "pentagon": 4, "cube": 3,
                    "sphere": 3, "pyramid": 2, "cylinder": 2, "cone": 2, "trapezoid": 1, "rhombus": 1
                },
                validSet: ["circle","square","triangle","rectangle","oval","ellipse","star","diamond","rhombus","heart","hexagon","octagon","pentagon","heptagon","nonagon","decagon","dodecagon","polygon","cube","sphere","pyramid","cylinder","cone","prism","rectangular prism","triangular prism","trapezoid","parallelogram","quadrilateral","crescent","semicircle","half circle","ring","annulus","torus","tetrahedron","octahedron","dodecahedron","icosahedron","cross","plus","arrow","spiral","helix","wave","line","dot","point","arc","sector","segment"]
            },
            {
                prompt: "A TYPE OF BUG OR INSECT",
                common: {
                    "ant": 34, "bee": 24, "butterfly": 18, "spider": 16, "ladybug": 12, "mosquito": 10,
                    "fly": 9, "beetle": 8, "grasshopper": 6, "cockroach": 5, "wasp": 4, "dragonfly": 3,
                    "caterpillar": 3, "moth": 2, "cricket": 2, "firefly": 2, "mantis": 1, "praying mantis": 1
                }, validSet: ["ant","bee","honeybee","bumblebee","butterfly","spider","ladybug","mosquito","fly","housefly","beetle","grasshopper","cockroach","roach","wasp","hornet","yellowjacket","dragonfly","damselfly","caterpillar","moth","cricket","firefly","lightning bug","mantis","praying mantis","centipede","millipede","worm","earthworm","termite","flea","tick","lice","gnat","cicada","locust","katydid","walking stick","stick bug","stink bug","june bug","scarab","weevil","aphid","earwig","silverfish","bedbug","tarantula","scorpion","pill bug","roly poly"]
            },
            {
                prompt: "A TYPE OF TREE",
                common: {
                    "oak": 38, "pine": 26, "maple": 20, "palm": 14, "palm tree": 14, "birch": 10,
                    "willow": 8, "weeping willow": 8, "redwood": 6, "apple tree": 5, "cherry blossom": 4,
                    "spruce": 3, "cedar": 3, "elm": 2, "fir": 2, "sequoia": 2, "magnolia": 1, "ash": 1
                },
                validSet: ["oak","pine","maple","palm","palm tree","birch","willow","weeping willow","redwood","apple tree","cherry blossom","sakura","spruce","cedar","elm","fir","douglas fir","sequoia","giant sequoia","magnolia","ash","beech","poplar","aspen","sycamore","hickory","walnut","chestnut","cypress","juniper","hemlock","mahogany","teak","ebony","banyan","baobab","eucalyptus","acacia","bamboo","dogwood","hawthorn","hazel","holly","larch","linden","locust","mulberry","olive","peach tree","pear tree","pecan","plum tree","yew","bonsai","mangrove"]
            },
            {
                prompt: "A CANDY OR CHOCOLATE BAR",
                common: {"snickers": 34, "m&ms": 24, "reese's": 20, "reeses": 20, "skittles": 16, "kit kat": 14,
                    "twix": 12, "hershey's": 10, "hersheys": 10, "starburst": 8, "gummy bears": 6,
                    "milky way": 5, "butterfinger": 4, "sour patch kids": 4, "3 musketeers": 3, "nerds": 2,
                    "jolly rancher": 2, "twizzlers": 2, "swedish fish": 1, "tootsie roll": 1
                },
                validSet: ["snickers","m&ms","reese's","reeses","reese's pieces","skittles","kit kat","twix","hershey's","hersheys","starburst","gummy bears","gummy worms","milky way","butterfinger","sour patch kids","3 musketeers","three musketeers","nerds","jolly rancher","twizzlers","swedish fish","tootsie roll","tootsie pop","lollipop","sucker","dum dums","airheads","laffy taffy","warheads","jawbreaker","gobstopper","mike and ike","hot tamales","junior mints","milk duds","whoppers","maltesers","crunch","nestle crunch","baby ruth","payday","almond joy","mounds","york peppermint patty","andes mints","toblerone","ferrero rocher","lindor","cadbury","smarties","sweetarts","pixy stix","fun dip","ring pop","push pop","pez","cotton candy","licorice","caramel","fudge","taffy","peppermint","candy cane"]
            },
            {
                prompt: "SOMETHING IN A BATHROOM",
                common: {
                    "toilet": 42, "shower": 28, "sink": 24, "toothbrush": 18, "toothpaste": 14, "towel": 12,
                    "bathtub": 10, "tub": 10, "soap": 9, "mirror": 8, "toilet paper": 7, "shampoo": 6,
                    "conditioner": 4, "rug": 2, "bath mat": 2, "floss": 2, "razor": 1, "hairdryer": 1
                },validSet: ["toilet","shower","sink","toothbrush","toothpaste","towel","hand towel","bath towel","bathtub","tub","soap","hand soap","body wash","mirror","toilet paper","shampoo","conditioner","rug","bath mat","floss","dental floss","mouthwash","razor","shaving cream","hairdryer","blow dryer","comb","brush","hairbrush","deodorant","perfume","cologne","lotion","moisturizer","sunscreen","q-tips","cotton swabs","cotton balls","bandages","band-aid","medicine cabinet","scale","plunger","toilet brush","trash can","hamper","shower curtain","faucet","drain","loofah","sponge","washcloth","robe","bathrobe","tissue","tissues","kleenex"]
            },
            {
                prompt: "A TYPE OF CHEESE",
                common: {
                    "cheddar": 42, "mozzarella": 26, "swiss": 16, "american": 14, "parmesan": 12,
                    "provolone": 8, "brie": 6, "gouda": 6, "blue cheese": 5, "feta": 5, "pepper jack": 4,
                    "monterey jack": 3, "colby jack": 3, "ricotta": 2, "cream cheese": 2, "goat cheese": 2,
                    "camembert": 1, "gruyere": 1, "manchego": 1, "gorgonzola": 1
                },
                validSet: ["cheddar","mozzarella","swiss","american","parmesan","parmigiano reggiano","provolone","brie","gouda","blue cheese","bleu cheese","feta","pepper jack","monterey jack","colby","colby jack","ricotta","cream cheese","goat cheese","chevre","camembert","gruyere","manchego","gorgonzola","roquefort","stilton","havarti","muenster","asiago","romano","pecorino","mascarpone","cottage cheese","string cheese","velveeta","nacho cheese","halloumi","paneer","queso fresco","cotija","oaxaca","edam","emmental","fontina","jarlsberg","limburger","neufchatel","raclette","taleggio"]
            },
            {
                prompt: "SOMETHING YOU FIND AT A CIRCUS OR CARNIVAL",
                common: {"clown": 38, "clowns": 38, "elephant": 20, "tent": 16, "ferris wheel": 14, "lion": 12,
                    "popcorn": 10, "cotton candy": 9, "acrobat": 7, "acrobats": 7, "trapeze": 6, "juggler": 5,
                    "roller coaster": 4, "ringmaster": 4, "tightrope": 3, "tiger": 3, "balloon": 2, "carousel": 2
                },
                validSet: ["clown","clowns","elephant","tent","big top","ferris wheel","lion","popcorn","cotton candy","acrobat","acrobats","trapeze","juggler","roller coaster","ringmaster","tightrope","tightrope walker","tiger","balloon","balloons","balloon animals","carousel","merry-go-round","bumper cars","funnel cake","corn dog","fried dough","candy apple","caramel apple","tickets","prizes","stuffed animals","ring toss","darts","dunk tank","funhouse","house of mirrors","fortune teller","magician","fire breather","sword swallower","unicycle","stilt walker","cannon","human cannonball","monkey","horse","seal","strongman","mime"]
            },
            {
                prompt: "A PART OF A CAR",
                common: {
                    "wheel": 32, "wheels": 32, "tire": 28, "tires": 28, "engine": 22, "steering wheel": 18,
                    "door": 14, "doors": 14, "window": 10, "windshield": 9, "seat": 8, "seats": 8,
                    "brake": 7, "brakes": 7, "headlight": 6, "headlights": 6, "trunk": 5, "hood": 5,
                    "mirror": 4, "bumper": 3, "exhaust": 2, "battery": 2, "radio": 2, "dashboard": 1
                }, validSet: ["wheel","wheels","tire","tires","engine","motor","steering wheel","door","doors","window","windows","windshield","seat","seats","seatbelt","brake","brakes","brake pedal","gas pedal","accelerator","clutch","headlight","headlights","taillight","blinker","turn signal","trunk","hood","mirror","rearview mirror","side mirror","bumper","fender","grille","exhaust","muffler","tailpipe","battery","alternator","radiator","transmission","gearshift","shifter","dashboard","speedometer","odometer","fuel gauge","radio","stereo","horn","airbag","glove box","sun visor","wiper","windshield wipers","roof","sunroof","hubcap","rim","axle","suspension","spark plug","carburetor","catalytic converter","gas tank"]
            },
            {
                prompt: "A TYPE OF PASTA",
                common: {
                    "spaghetti": 44, "macaroni": 22, "penne": 16, "fettuccine": 12, "lasagna": 10,
                    "ravioli": 9, "linguine": 7, "rotini": 5, "rigatoni": 5, "bowtie": 4, "farfalle": 4,
                    "tortellini": 4, "angel hair": 3, "shells": 2, "gnocchi": 2, "orzo": 2, "fusilli": 1, "ziti": 1
                },
                validSet: ["spaghetti","macaroni","elbow macaroni","penne","fettuccine","lasagna","ravioli","linguine","rotini","rigatoni","bowtie","farfalle","tortellini","angel hair","capellini","shells","conchiglie","gnocchi","orzo","fusilli","ziti","cannelloni","manicotti","tagliatelle","pappardelle","bucatini","cavatappi","gemelli","orecchiette","paccheri","radiatori","vermicelli","ramen","udon","soba","egg noodles","couscous","pastina","acini di pepe","ditalini","campanelle","casarecce","strozzapreti","trofie"]
            },
            { prompt: "SOMETHING IN A CLASSROOM",
                common: {
                    "desk": 38, "chair": 26, "whiteboard": 20, "chalkboard": 18, "teacher": 16, "pencil": 14,
                    "book": 12, "books": 12, "clock": 8, "computer": 7, "paper": 6, "backpack": 5,
                    "eraser": 4, "marker": 4, "ruler": 3, "globe": 2, "map": 2, "projector": 2, "stapler": 1
                },
                validSet: ["desk","desks","chair","chairs","whiteboard","chalkboard","blackboard","teacher","student","students","pencil","pen","book","books","textbook","notebook","clock","computer","laptop","chromebook","tablet","paper","backpack","eraser","marker","dry erase marker","chalk","ruler","globe","map","projector","smartboard","stapler","scissors","glue","glue stick","tape","crayons","colored pencils","markers","highlighter","folder","binder","pencil sharpener","trash can","recycling bin","bookshelf","cubby","locker","flag","poster","calendar","bulletin board","bell","calculator","protractor","compass"]
            },
            {
                prompt: "A TOY OR CHILDHOOD PLAYTHING",
                common: {
                    "lego": 32, "legos": 32, "doll": 24, "barbie": 20, "action figure": 14, "ball": 12,
                    "teddy bear": 11, "stuffed animal": 10, "hot wheels": 8, "toy car": 8, "nerf gun": 6,
                    "yo-yo": 4, "yoyo": 4, "slinky": 4, "rubik's cube": 3, "play-doh": 3, "puzzle": 3,
                    "kite": 2, "frisbee": 2, "jump rope": 2, "blocks": 2, "rc car": 1, "tamagotchi": 1
                }, validSet: ["lego","legos","doll","dolls","barbie","action figure","ball","teddy bear","stuffed animal","plushie","hot wheels","toy car","matchbox car","nerf gun","nerf","yo-yo","yoyo","slinky","rubik's cube","rubiks cube","play-doh","playdoh","puzzle","jigsaw puzzle","kite","frisbee","jump rope","blocks","building blocks","wooden blocks","rc car","remote control car","train set","toy train","dollhouse","easy bake oven","etch a sketch","mr potato head","furby","beanie baby","pokemon cards","transformer","transformers","gi joe","my little pony","polly pocket","cabbage patch kid","american girl doll","squishmallow","fidget spinner","slime","water gun","super soaker","hula hoop","pogo stick","tricycle","bicycle","scooter","skateboard","marbles","jacks","top","spinning top","rocking horse","bop it","simon"]
            },
            {
                prompt: "A NATURAL DISASTER",
                common: {
                    "earthquake": 34, "tornado": 28, "hurricane": 24, "tsunami": 16, "flood": 14,
                    "volcano": 10, "volcanic eruption": 10, "wildfire": 8, "fire": 8, "blizzard": 5,
                    "landslide": 4, "avalanche": 4, "drought": 3, "typhoon": 3, "mudslide": 2, "sinkhole": 1,
                    "dust storm": 1, "heatwave": 1, "cyclone": 1
                },
                validSet: ["earthquake","tornado","twister","hurricane","tsunami","tidal wave","flood","flash flood","volcano","volcanic eruption","eruption","wildfire","forest fire","fire","blizzard","ice storm","snowstorm","landslide","avalanche","drought","typhoon","cyclone","mudslide","sinkhole","dust storm","sandstorm","heatwave","cold wave","hailstorm","thunderstorm","lightning storm","monsoon","famine","plague","meteor impact","asteroid impact","solar storm","geomagnetic storm","limnic eruption"]
            },
            {
                prompt: "A COUNTRY IN ASIA",
                common: {
                    "china": 36, "japan": 32, "india": 22, "south korea": 14, "korea": 14, "thailand": 10,
                    "vietnam": 8, "russia": 7, "philippines": 6, "indonesia": 5, "taiwan": 4, "singapore": 3,
                    "malaysia": 3, "pakistan": 2, "north korea": 2, "saudi arabia": 2, "israel": 1, "nepal": 1
                },
                validSet: ["china","japan","india","south korea","north korea","korea","thailand","vietnam","russia","philippines","indonesia","taiwan","singapore","malaysia","pakistan","bangladesh","sri lanka","nepal","bhutan","myanmar","burma","cambodia","laos","mongolia","kazakhstan","uzbekistan","turkmenistan","kyrgyzstan","tajikistan","afghanistan","iran","iraq","saudi arabia","united arab emirates","uae","qatar","kuwait","bahrain","oman","yemen","jordan","lebanon","syria","israel","palestine","turkey","georgia","armenia","azerbaijan","maldives","brunei","timor-leste","east timor"]
            },
            {
                prompt: "SOMETHING YOU DO IN THE MORNING",
                common: {
                    "brush teeth": 36, "brush my teeth": 36, "eat breakfast": 28, "breakfast": 28,
                    "wake up": 24, "shower": 18, "take a shower": 18, "drink coffee": 14, "coffee": 14,
                    "get dressed": 12, "wash face": 8, "stretch": 5, "yawn": 4, "check phone": 4,
                    "exercise": 3, "work out": 3, "make bed": 3, "comb hair": 2, "shave": 1, "meditate": 1
                },
                validSet: ["brush teeth","brush my teeth","eat breakfast","breakfast","wake up","get up","get out of bed","shower","take a shower","drink coffee","coffee","drink tea","drink water","get dressed","put on clothes","wash face","wash my face","stretch","yawn","check phone","check my phone","exercise","work out","run","jog","walk the dog","feed the dog","feed the cat","make bed","make my bed","comb hair","brush hair","shave","put on makeup","skincare","meditate","pray","read the news","watch tv","listen to music","go to school","go to work","commute","drive","take a bath","floss","use mouthwash","use the bathroom","pee","poop","snooze","hit snooze"]
            },
            {
                prompt: "A GEMSTONE OR PRECIOUS METAL",
                common: {
                    "diamond": 40, "gold": 30, "ruby": 18, "emerald": 16, "sapphire": 14, "silver": 12,
                    "amethyst": 6, "pearl": 6, "topaz": 4, "opal": 4, "platinum": 4, "jade": 3,
                    "turquoise": 3, "garnet": 2, "quartz": 2, "obsidian": 2, "amber": 1, "onyx": 1
                },
                validSet: ["diamond","gold","ruby","emerald","sapphire","silver","amethyst","pearl","topaz","opal","platinum","jade","turquoise","garnet","quartz","rose quartz","obsidian","amber","onyx","aquamarine","peridot","citrine","tanzanite","alexandrite","moonstone","sunstone","lapis lazuli","malachite","agate","jasper","carnelian","tourmaline","zircon","spinel","morganite","kunzite","bloodstone","chrysocolla","fluorite","hematite","kyanite","labradorite","lepidolite","pyrite","rhodonite","selenite","sodalite","tigers eye","tiger's eye","copper","bronze","titanium","palladium","rhodium","iridium","osmium"]
            }
        ];

        // Levenshtein distance for forgiving fuzzy autocorrect
        function levenshtein(a, b) {
            const matrix = Array.from({ length: b.length + 1 }, (_, i) => [i]);
            for (let j = 0; j <= a.length; j++) matrix[0][j] = j;
            for (let i = 1; i <= b.length; i++) {
                for (let j = 1; j <= a.length; j++) {
                    if (b.charAt(i - 1) === a.charAt(j - 1)) {
                        matrix[i][j] = matrix[i - 1][j - 1];
                    } else {
                        matrix[i][j] = Math.min(
                            matrix[i - 1][j - 1] + 1,
                            matrix[i][j - 1] + 1,
                            matrix[i - 1][j] + 1
                        );
                    }
                }
            }
            return matrix[b.length][a.length];
        }

        function evaluateKrillionAnswer(rawInput, questionObj) {
            const cleaned = rawInput.toLowerCase().trim().replace(/[^a-z0-9\s'-]/g, '');
            if (!cleaned) {
                return { valid: false, corrected: "", pct: 100, wasCorrected: false };
            }

            // 1. Exact match in validSet or common
            if (questionObj.validSet.includes(cleaned) || questionObj.common[cleaned]) {
                const pct = questionObj.common[cleaned] || 1;
                return { valid: true, corrected: cleaned.toUpperCase(), pct, wasCorrected: false };
            }

            // 2. Fuzzy autocorrect against validSet
            let bestMatch = null;
            let bestDist = Infinity;
            for (const candidate of questionObj.validSet) {
                const dist = levenshtein(cleaned, candidate);
                const maxAllowed = candidate.length <= 4 ? 1 : (candidate.length <= 7 ? 2 : 3);
                if (dist <= maxAllowed && dist < bestDist) {
                    bestDist = dist;
                    bestMatch = candidate;
                }
            }

            if (bestMatch) {
                const pct = questionObj.common[bestMatch] || 1;
                return {
                    valid: true,
                    corrected: bestMatch.toUpperCase(),
                    pct,
                    wasCorrected: bestMatch !== cleaned
                };
            }

            // 3. Creative fallback check for plausible multi-word or obscure nouns (if >= 3 chars & vowel)
            if (cleaned.length >= 4 && /[aeiouy]/.test(cleaned) && !/(asdf|qwer|zxcv|aaaa|bbbb|xxxx|test|idk|none|nothing)/.test(cleaned)) {
                return { valid: true, corrected: cleaned.toUpperCase(), pct: 2, wasCorrected: false };
            }

            return { valid: false, corrected: cleaned.toUpperCase(), pct: 100, wasCorrected: false };
        }

        // ============================================================================
        // 3. 10-STAGE BOSS CONFIGURATION (HARDER SCALING + 15% LESS BOSS DAMAGE)
        // ============================================================================
        const BOSS_STAGES = [
            {
                stage: 1,
                name: "APPRENTICE MAID REX",
                subtitle: "STAGE 1/10 - SPECIAL: SOAP BUBBLE BARRAGE",
                specialName: "SOAP BUBBLE BARRAGE",
                maxHp: 460,
                scaleColor: "#38b764",
                scaleDark: "#1e6f50",
                accentColor: "#ff5ca8",
                speed: 96,
                attackInterval: 1.85,
                damageMult: 0.95,
                crown: false,
                orbitCups: 1,
                auraColor: null
            },
            {
                stage: 2,
                name: "PARLOR MAID RAPTOR-REX",
                subtitle: "STAGE 2/10 - SPECIAL: BROOMSTICK CHARGE",
                specialName: "BROOMSTICK CHARGE",
                maxHp: 680,
                scaleColor: "#29adff",
                scaleDark: "#1d2b53",
                accentColor: "#00f0ff",
                speed: 112,
                attackInterval: 1.72,
                damageMult: 1.05,
                crown: false,
                orbitCups: 2,
                auraColor: "rgba(0, 240, 255, 0.14)"
            },
            {
                stage: 3,
                name: "KITCHEN MAID ALLOSAUR",
                subtitle: "STAGE 3/10 - SPECIAL: BOILING KETTLE GEYSER",
                specialName: "BOILING KETTLE GEYSER",
                maxHp: 940,
                scaleColor: "#ff77a8",
                scaleDark: "#7e2553",
                accentColor: "#ffd700",
                speed: 124,
                attackInterval: 1.60,
                damageMult: 1.12,
                crown: false,
                orbitCups: 2,
                auraColor: "rgba(255, 119, 168, 0.16)"
            },
            {
                stage: 4,
                name: "HEAD MAID TYRANNOSAUR",
                subtitle: "STAGE 4/10 - SPECIAL: SILVERWARE FAN VOLLEY",
                specialName: "SILVERWARE FAN VOLLEY",
                maxHp: 1250,
                scaleColor: "#ff6b35",
                scaleDark: "#7c2d12",
                accentColor: "#e2e8f0",
                speed: 136,
                attackInterval: 1.48,
                damageMult: 1.20,
                crown: true,
                orbitCups: 3,
                auraColor: "rgba(255, 107, 53, 0.18)"
            },
            {
                stage: 5,
                name: "BUTLER-GENERAL SPINOSAUR",
                subtitle: "STAGE 5/10 - SPECIAL: CHANDELIER SHATTER",
                specialName: "CHANDELIER SHATTER",
                maxHp: 1620,
                scaleColor: "#a855f7",
                scaleDark: "#4c1d95",
                accentColor: "#ffd700",
                speed: 148,
                attackInterval: 1.36,
                damageMult: 1.28,
                crown: true,
                orbitCups: 3,
                auraColor: "rgba(168, 85, 247, 0.22)"
            },
            {
                stage: 6,
                name: "ROYAL VALET GIGANOTOSAUR",
                subtitle: "STAGE 6/10 - SPECIAL: POLISH MIRROR BEAM",
                specialName: "POLISH MIRROR BEAM",
                maxHp: 2050,
                scaleColor: "#06b6d4",
                scaleDark: "#164e63",
                accentColor: "#38bdf8",
                speed: 160,
                attackInterval: 1.25,
                damageMult: 1.36,
                crown: true,
                orbitCups: 4,
                auraColor: "rgba(6, 182, 212, 0.24)"
            },
            {
                stage: 7,
                name: "GRAND ARCH-MAID OMEGA REX",
                subtitle: "STAGE 7/10 - SPECIAL: VACUUM VORTEX PULL",
                specialName: "VACUUM VORTEX PULL",
                maxHp: 2550,
                scaleColor: "#ec4899",
                scaleDark: "#831843",
                accentColor: "#c084fc",
                speed: 172,
                attackInterval: 1.15,
                damageMult: 1.45,
                crown: true,
                orbitCups: 4,
                auraColor: "rgba(236, 72, 153, 0.26)"
            },
            {
                stage: 8,
                name: "CELESTIAL EMPRESS REX",
                subtitle: "STAGE 8/10 - SPECIAL: SOLAR DUSTER NOVA",
                specialName: "SOLAR DUSTER NOVA",
                maxHp: 3150,
                scaleColor: "#f59e0b",
                scaleDark: "#78350f",
                accentColor: "#fef08a",
                speed: 184,
                attackInterval: 1.05,
                damageMult: 1.55,
                crown: true,
                orbitCups: 5,
                auraColor: "rgba(245, 158, 11, 0.30)"
            },
            {
                stage: 9,
                name: "CHRONO-MAID APEX REX",
                subtitle: "STAGE 9/10 - SPECIAL: GRANDFATHER CLOCK SPIRAL",
                specialName: "GRANDFATHER CLOCK SPIRAL",
                maxHp: 3820,
                scaleColor: "#10b981",
                scaleDark: "#064e3b",
                accentColor: "#39ff14",
                speed: 194,
                attackInterval: 0.96,
                damageMult: 1.65,
                crown: true,
                orbitCups: 5,
                auraColor: "rgba(57, 255, 20, 0.32)"
            },
            {
                stage: 10,
                name: "VOID-MAID OVERLORD EX",
                subtitle: "STAGE 10/10 - SPECIAL: MINI MAID RAPTOR SQUAD",
                specialName: "MINI MAID RAPTOR SQUAD",
                maxHp: 4600,
                scaleColor: "#1e1633",
                scaleDark: "#090512",
                accentColor: "#ff2a4b",
                speed: 205,
                attackInterval: 0.88,
                damageMult: 1.75,
                crown: true,
                orbitCups: 6,
                auraColor: "rgba(255, 42, 75, 0.38)"
            }
        ];

        // ============================================================================
        // 4. PERMANENT POWERS (1-PER-GAME ON <25% ANSWERS) & TEMPORARY BUFFS
        // ============================================================================
        const PERMANENT_POWERS = {
            FLAME: {
                id: 'FLAME',
                label: 'FLAME HANDGUNS (1.5X DMG)',
                shortName: 'FLAME HANDGUNS',
                damageMult: 1.5,
                bulletColor: '#ff6b1a',
                desc: 'PERMANENT POWER: DUAL FLAME HANDGUNS (1.5X DAMAGE)!'
            },
            LIGHTNING: {
                id: 'LIGHTNING',
                label: 'LIGHTNING (1s STUN / 10s)',
                shortName: 'LIGHTNING STUN',
                damageMult: 1.0,
                bulletColor: '#ffe600',
                desc: 'PERMANENT POWER: LIGHTNING STUNS DINO FOR 1s EVERY 10s!'
            },
            ICE: {
                id: 'ICE',
                label: 'ICE FROST (3s SLOW / 10s)',
                shortName: 'ICE FREEZE',
                damageMult: 1.0,
                bulletColor: '#7dd3fc',
                desc: 'PERMANENT POWER: ICE SLOWS DINO FOR 3s EVERY 10s!'
            },
            SLOWDOWN: {
                id: 'SLOWDOWN',
                label: 'CHRONO SLOW (50% DINO/SHOTS)',
                shortName: 'CHRONO SLOW-DOWN',
                damageMult: 1.0,
                bulletColor: '#d946ef',
                desc: 'ULTRA RARE (1-IN-50): 50% SLOW-DOWN ON DINO & PROJECTILES!'
            }
        };

        function rollPermanentPower() {
            // 1 in 50 chance for Chrono Slow-Down
            if (Math.random() < (1 / 50)) {
                return PERMANENT_POWERS.SLOWDOWN;
            }
            const pool = [PERMANENT_POWERS.FLAME, PERMANENT_POWERS.LIGHTNING, PERMANENT_POWERS.ICE];
            return pool[Math.floor(Math.random() * pool.length)];
        }

        function computeTriviaReward(evalResult) {
            if (!evalResult.valid) {
                return {
                    title: "INVALID OR GIBBERISH ANSWER!",
                    desc: "NO BUFF GRANTED! +10s SHIELD WEAKNESS.",
                    heal: 0,
                    buff: null,
                    newPower: null
                };
            }

            const pct = evalResult.pct;
            let newPower = null;

            // First 1% rarity answer guarantees a permanent power (100% chance); other <10% answers have a 25% chance
            if (!player.permanentPower && pct < 10) {
                const isFirstOnePercent = (pct === 1 && !player.usedGuaranteedOnePct);
                if (isFirstOnePercent) {
                    player.usedGuaranteedOnePct = true;
                    newPower = rollPermanentPower();
                } else if (Math.random() < 0.25) {
                    newPower = rollPermanentPower();
                }
            }

            const hasFlamePower = (player.permanentPower && player.permanentPower.id === 'FLAME') || (newPower && newPower.id === 'FLAME');
            const spreadDuration = hasFlamePower ? 5 : 10;

            if (pct <= 12) {
                return {
                    title: `LEGENDARY! ONLY ${pct}% SAID "${evalResult.corrected}"!`,
                    desc: newPower
                        ? `${newPower.desc} + FULL HEAL (50 HP)!`
                        : `TRIPLE PLASMA CANNON (${spreadDuration}s, -10% DMG/BOLT) + HEAL 50 HP!`,
                    heal: 50,
                    buff: { name: "TRIPLE PLASMA", spread: 3, fireRate: 0.17, damage: 15, color: "#ffd700", timer: spreadDuration },
                    newPower
                };
            } else if (pct >= 13 && pct <= 50) {
                return {
                    title: `SOLID PICK! ${pct}% SAID "${evalResult.corrected}"!`,
                    desc: newPower
                        ? `${newPower.desc} + HEAL 30 HP!`
                        : `DOUBLE SHOT (${spreadDuration}s, -5% DMG/BOLT) + HEAL 30 HP!`,
                    heal: 30,
                    buff: { name: "DOUBLE SHOT", spread: 2, fireRate: 0.18, damage: 15, color: "#ff2a85", timer: spreadDuration },
                    newPower
                };
            } else {
                return {
                    title: `COMMON ANSWER (${pct}%): "${evalResult.corrected}"`,
                    desc: "STANDARD BLASTER + SMALL HEAL (12 HP).",
                    heal: 12,
                    buff: null,
                    newPower: null
                };
            }
        }

        // ============================================================================
        // 5. GAME ENGINE STATE & ENTITIES
        // ============================================================================
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        const GAME_W = 800;
        const GAME_H = 450;
        const GROUND_Y = 392;
        const KRILLION_INTERVAL = 15.0;

        let gameState = 'START'; // START | PLAYING | QUIZ | GAMEOVER | VICTORY
        let lastTimestamp = 0;
        let screenShake = 0;

        let stats = {
            survivalTime: 0,
            stagesCleared: 0,
            rarestPct: null,
            score: 0
        };

        let krillionTimer = KRILLION_INTERVAL;
        let quizCountdown = 20.0;
        let currentQuestion = null;
        let usedQuestionIndices = new Set();

        let stageTransitionTimer = 0;
        let stageBannerData = null;

        const keys = {};

        const player = {
            x: 130,
            y: GROUND_Y,
            w: 28,
            h: 42,
            vx: 0,
            vy: 0,
            speed: 235,
            jumpForce: -470,
            gravity: 1180,
            onGround: true,
            facing: 1,
            hp: 100,
            maxHp: 100,
            invulnTimer: 0,
            shootCooldown: 0,
            dashTimer: 0,
            dashCooldown: 0,
            animFrame: 0,
            activeBuff: null,
            permanentPower: null,
            powerCycleTimer: 0
        };

        const boss = {
            stageIndex: 0,
            x: 560,
            y: GROUND_Y,
            w: 116,
            h: 132,
            vx: -96,
            vy: 0,
            patrolDir: -1,
            patrolTimer: 1.8,
            facing: -1,
            hp: 460,
            maxHp: 460,
            attackTimer: 1.4,
            jumpCooldown: 6.5,
            attackCounter: 0,
            isJumping: false,
            chargeTimer: 0,
            vortexTimer: 0,
            stunTimer: 0,
            iceTimer: 0,
            hitFlash: 0,
            animTimer: 0,
            orbitAngle: 0
        };

        let playerBullets = [];
        let bossProjectiles = [];
        let miniRaptors = [];
        let particles = [];
        let floatingTexts = [];

        // DOM References
        const hudOverlay = document.getElementById('hud-overlay');
        const hudPlayerHpText = document.getElementById('hud-player-hp-text');
        const hudPlayerHpBar = document.getElementById('hud-player-hp-bar');
        const hudBuffTag = document.getElementById('hud-buff-tag');
        const hudPowerTag = document.getElementById('hud-power-tag');
        const hudKrillionTimer = document.getElementById('hud-krillion-timer');
        const hudScore = document.getElementById('hud-score');
        const hudStageBadge = document.getElementById('hud-stage-badge');
        const hudBossName = document.getElementById('hud-boss-name');
        const hudBossHpBar = document.getElementById('hud-boss-hp-bar');
        const hudBossHpText = document.getElementById('hud-boss-hp-text');

        const startScreen = document.getElementById('start-screen');
        const quizOverlay = document.getElementById('quiz-overlay');
        const quizTimerEl = document.getElementById('quiz-timer');
        const quizPromptEl = document.getElementById('quiz-prompt');
        const quizInputArea = document.getElementById('quiz-input-area');
        const quizInput = document.getElementById('quiz-input');
        const quizResultArea = document.getElementById('quiz-result-area');
        const quizAutocorrectNote = document.getElementById('quiz-autocorrect-note');
        const quizRarityDisplay = document.getElementById('quiz-rarity-display');
        const quizPctLabel = document.getElementById('quiz-pct-label');
        const quizRarityBar = document.getElementById('quiz-rarity-bar');
        const quizRewardDesc = document.getElementById('quiz-reward-desc');
        const endScreen = document.getElementById('end-screen');

        // ============================================================================
        // 6. RESIZE & INPUT HANDLERS
        // ============================================================================
        function resizeCanvas() {
            const rect = canvas.parentElement.getBoundingClientRect();
            canvas.width = GAME_W;
            canvas.height = GAME_H;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        window.addEventListener('keydown', (e) => {
            if (gameState === 'QUIZ') {
                if (e.key === 'Enter') {
                    e.preventDefault();
                    if (!quizInputArea.classList.contains('hidden')) {
                        submitTriviaAnswer();
                    } else {
                        resumeFromQuiz();
                    }
                }
                return;
            }
            keys[e.code] = true;
            if (['Space', 'ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.code)) {
                e.preventDefault();
            }
        });

        window.addEventListener('keyup', (e) => {
            keys[e.code] = false;
        });

        // Mobile touch bindings
        function bindHoldButton(id, code) {
            const el = document.getElementById(id);
            if (!el) return;
            const start = (e) => { e.preventDefault(); sfx.init(); keys[code] = true; };
            const end = (e) => { e.preventDefault(); keys[code] = false; };
            el.addEventListener('touchstart', start, { passive: false });
            el.addEventListener('touchend', end, { passive: false });
            el.addEventListener('mousedown', start);
            el.addEventListener('mouseup', end);
            el.addEventListener('mouseleave', end);
        }
        bindHoldButton('m-left', 'ArrowLeft');
        bindHoldButton('m-right', 'ArrowRight');
        bindHoldButton('m-jump', 'ArrowUp');
        bindHoldButton('m-dash', 'ShiftLeft');
        bindHoldButton('m-fire', 'Space');

        // ============================================================================
        // 7. HELPER SPAWNERS & STAGE PROGRESSION
        // ============================================================================
        function spawnParticles(x, y, color, count = 8, maxSpeed = 120) {
            for (let i = 0; i < count; i++) {
                const ang = Math.random() * Math.PI * 2;
                const spd = 25 + Math.random() * maxSpeed;
                particles.push({
                    x,
                    y,
                    vx: Math.cos(ang) * spd,
                    vy: Math.sin(ang) * spd - 30,
                    size: 2 + Math.random() * 3,
                    color,
                    life: 0.35 + Math.random() * 0.3,
                    maxLife: 0.65
                });
            }
        }

        function spawnFloatingText(x, y, text, color = '#ffffff') {
            floatingTexts.push({
                x,
                y,
                text,
                color,
                life: 1.05
            });
        }

        function configureBossStage(idx) {
            const cfg = BOSS_STAGES[idx];
            boss.stageIndex = idx;
            boss.maxHp = cfg.maxHp;
            boss.hp = cfg.maxHp;
            boss.x = 560;
            boss.y = GROUND_Y;
            boss.vx = -cfg.speed;
            boss.vy = 0;
            boss.patrolDir = -1;
            boss.patrolTimer = 1.8;
            boss.facing = -1;
            boss.attackTimer = 1.25;
            boss.jumpCooldown = 5.0 + Math.random() * 5.0;
            boss.attackCounter = 0;
            boss.isJumping = false;
            boss.chargeTimer = 0;
            boss.vortexTimer = 0;
            boss.stunTimer = 0;
            boss.iceTimer = 0;
            boss.hitFlash = 0;
        }

        function startNewGame() {
            sfx.init();
            stats = { survivalTime: 0, stagesCleared: 0, rarestPct: null, score: 0 };
            krillionTimer = KRILLION_INTERVAL;
            usedQuestionIndices.clear();
            playerBullets = [];
            bossProjectiles = [];
            miniRaptors = [];
            particles = [];
            floatingTexts = [];
            stageTransitionTimer = 0;
            stageBannerData = null;

            player.x = 130;
            player.y = GROUND_Y;
            player.vx = 0;
            player.vy = 0;
            player.onGround = true;
            player.facing = 1;
            player.maxHp = 100;
            player.hp = 100;
            player.invulnTimer = 0;
            player.shootCooldown = 0;
            player.dashTimer = 0;
            player.dashCooldown = 0;
            player.activeBuff = null;
            player.permanentPower = null;
            player.powerCycleTimer = 0;
            player.usedGuaranteedOnePct = false;

            configureBossStage(0);

            startScreen.classList.add('hidden');
            endScreen.classList.add('hidden');
            quizOverlay.classList.add('hidden');
            hudOverlay.classList.remove('hidden');

            gameState = 'PLAYING';
            updateHUD();
        }

        function triggerStageTransition(nextStageIdx) {
            stats.stagesCleared = nextStageIdx;
            stats.score += 1000 * nextStageIdx;
            sfx.stageClear();

            // +10% permanent max health increase after each round
            const oldMax = player.maxHp;
            player.maxHp = Math.round(player.maxHp * 1.10);
            const bonusHp = player.maxHp - oldMax;
            player.hp = Math.min(player.maxHp, player.hp + bonusHp + 20);

            configureBossStage(nextStageIdx);
            bossProjectiles = [];
            miniRaptors = [];
            screenShake = 14;
            stageTransitionTimer = 2.6;
            const cfg = BOSS_STAGES[nextStageIdx];
            stageBannerData = {
                title: `STAGE ${cfg.stage} / 10`,
                subtitle: cfg.name,
                bonus: `+10% MAX HP (${player.maxHp} HP)!`
            };
            spawnParticles(boss.x + boss.w / 2, boss.y - boss.h / 2, cfg.accentColor, 28, 190);
        }

        function damagePlayer(rawAmount) {
            if (player.invulnTimer > 0 || player.dashTimer > 0) return;
            const dmg = Math.max(1, Math.round(rawAmount));
            player.hp = Math.max(0, player.hp - dmg);
            player.invulnTimer = 0.85;
            screenShake = 8;
            sfx.playerHurt();
            spawnParticles(player.x + player.w / 2, player.y - player.h / 2, '#ff2a85', 12, 130);
            spawnFloatingText(player.x + 10, player.y - player.h - 8, `-${dmg}`, '#ff2a85');

            if (player.hp <= 0) {
                triggerEndGame(false);
            }
        }

        function triggerEndGame(victory) {
            gameState = victory ? 'VICTORY' : 'GAMEOVER';
            hudOverlay.classList.add('hidden');
            quizOverlay.classList.add('hidden');
            endScreen.classList.remove('hidden');

            const titleEl = document.getElementById('end-title');
            const subEl = document.getElementById('end-subtitle');
            if (victory) {
                titleEl.textContent = 'VICTORY!';
                titleEl.className = 'text-2xl sm:text-4xl text-green-400 neon-text-cyan mb-2';
                subEl.textContent = 'YOU CONQUERED ALL 10 DINO MAID STAGES!';
                sfx.stageClear();
            } else {
                titleEl.textContent = 'GAME OVER';
                titleEl.className = 'text-2xl sm:text-4xl text-pink-500 neon-text-pink mb-2';
                subEl.textContent = `FALLEN ON STAGE ${boss.stageIndex + 1} (${BOSS_STAGES[boss.stageIndex].name})`;
            }

            document.getElementById('end-stages').textContent = `${victory ? 10 : stats.stagesCleared} / 10`;
            document.getElementById('end-time').textContent = `${stats.survivalTime.toFixed(1)}s`;
            document.getElementById('end-rarest').textContent = stats.rarestPct !== null ? `${stats.rarestPct}%` : 'N/A';
            document.getElementById('end-score').textContent = `${stats.score}`;
        }

        // ============================================================================
        // 8. KRILLION TRIVIA PAUSE FLOW
        // ============================================================================
        function triggerKrillionQuiz() {
            gameState = 'QUIZ';
            sfx.quizFreeze();

            if (usedQuestionIndices.size >= TRIVIA_BANK.length) {
                usedQuestionIndices.clear();
            }
            let idx;
            do {
                idx = Math.floor(Math.random() * TRIVIA_BANK.length);
            } while (usedQuestionIndices.has(idx));
            usedQuestionIndices.add(idx);
            currentQuestion = TRIVIA_BANK[idx];

            quizCountdown = 20.0;
            quizTimerEl.textContent = '20.0s';
            quizPromptEl.textContent = currentQuestion.prompt;
            quizInput.value = '';
            quizInputArea.classList.remove('hidden');
            quizResultArea.classList.add('hidden');
            quizOverlay.classList.remove('hidden');

            setTimeout(() => quizInput.focus(), 40);
        }

        function submitTriviaAnswer() {
            if (!quizResultArea.classList.contains('hidden')) return;
            const evalRes = evaluateKrillionAnswer(quizInput.value, currentQuestion);
            const reward = computeTriviaReward(evalRes);

            if (evalRes.valid) {
                stats.score += Math.max(100, (101 - evalRes.pct) * 15);
                if (stats.rarestPct === null || evalRes.pct < stats.rarestPct) {
                    stats.rarestPct = evalRes.pct;
                }
            }

            if (reward.heal > 0) {
                player.hp = Math.min(player.maxHp, player.hp + reward.heal);
            }
            if (reward.buff) {
                player.activeBuff = { ...reward.buff };
            }
            if (reward.newPower) {
                player.permanentPower = reward.newPower;
                player.powerCycleTimer = 0;
            }

            sfx.powerUp(evalRes.valid && (evalRes.pct < 10 || reward.newPower));

            quizAutocorrectNote.textContent = evalRes.wasCorrected
                ? `AUTOCORRECTED TO: "${evalRes.corrected}"`
                : (evalRes.valid ? `VERIFIED: "${evalRes.corrected}"` : 'UNRECOGNIZED ANSWER');
            quizRarityDisplay.textContent = reward.title;
            quizRewardDesc.textContent = reward.desc;

            // Animate the rarity percentage bar
            const displayPct = evalRes.valid ? Math.max(1, Math.min(100, evalRes.pct)) : 100;
            quizPctLabel.textContent = evalRes.valid ? `${displayPct}% OF PLAYERS` : 'INVALID (100%)';
            quizRarityBar.style.width = '0%';
            if (!evalRes.valid) {
                quizRarityBar.className = 'h-full bg-red-600 transition-all duration-500 ease-out';
            } else if (displayPct < 10) {
                quizRarityBar.className = 'h-full bg-gradient-to-r from-green-400 to-cyan-400 transition-all duration-500 ease-out';
            } else {
                quizRarityBar.className = 'h-full bg-gradient-to-r from-yellow-400 to-pink-500 transition-all duration-500 ease-out';
            }

            quizInputArea.classList.add('hidden');
            quizResultArea.classList.remove('hidden');

            requestAnimationFrame(() => {
                requestAnimationFrame(() => {
                    quizRarityBar.style.width = `${displayPct}%`;
                });
            });
            updateHUD();
        }

        function resumeFromQuiz() {
            quizOverlay.classList.add('hidden');
            krillionTimer = KRILLION_INTERVAL;
            player.invulnTimer = 1.0;
            gameState = 'PLAYING';
            lastTimestamp = performance.now();
        }

        document.getElementById('btn-start').addEventListener('click', startNewGame);
        document.getElementById('btn-restart').addEventListener('click', startNewGame);
        document.getElementById('btn-submit-answer').addEventListener('click', submitTriviaAnswer);
        document.getElementById('btn-resume-battle').addEventListener('click', resumeFromQuiz);

        // ============================================================================
        // 9. PLAYER, BOSS & PROJECTILE COMBAT SIMULATION
        // ============================================================================
        function updatePlayer(dt) {
            if (player.invulnTimer > 0) player.invulnTimer -= dt;
            if (player.shootCooldown > 0) player.shootCooldown -= dt;
            if (player.dashCooldown > 0) player.dashCooldown -= dt;

            if (player.activeBuff) {
                player.activeBuff.timer -= dt;
                if (player.activeBuff.timer <= 0) {
                    player.activeBuff = null;
                }
            }

            // Periodic Permanent Powers (Lightning 1s stun / 10s, Ice 3s slow / 10s)
            if (player.permanentPower) {
                const pid = player.permanentPower.id;
                if (pid === 'LIGHTNING' || pid === 'ICE') {
                    player.powerCycleTimer += dt;
                    if (player.powerCycleTimer >= 10.0) {
                        player.powerCycleTimer = 0;
                        if (pid === 'LIGHTNING') {
                            boss.stunTimer = 1.0;
                            sfx.powerUp(false);
                            spawnFloatingText(boss.x + boss.w / 2, boss.y - boss.h - 12, '⚡ STUNNED 1s!', '#ffe600');
                            spawnParticles(boss.x + boss.w / 2, boss.y - boss.h / 2, '#ffe600', 16, 150);
                        } else if (pid === 'ICE') {
                            boss.iceTimer = 3.0;
                            sfx.powerUp(false);
                            spawnFloatingText(boss.x + boss.w / 2, boss.y - boss.h - 12, '❄️ CHILLED 3s!', '#7dd3fc');
                            spawnParticles(boss.x + boss.w / 2, boss.y - boss.h / 2, '#7dd3fc', 16, 140);
                        }
                    }
                }
            }

            // Player horizontal movement & dash
            let moveDir = 0;
            if (keys['ArrowLeft'] || keys['KeyA']) moveDir -= 1;
            if (keys['ArrowRight'] || keys['KeyD']) moveDir += 1;

            if (moveDir !== 0) {
                player.facing = moveDir;
                player.animFrame += dt * 12;
            } else {
                player.animFrame = 0;
            }

            if ((keys['ShiftLeft'] || keys['ShiftRight'] || keys['KeyX']) && player.dashCooldown <= 0 && player.dashTimer <= 0) {
                player.dashTimer = 0.16;
                player.dashCooldown = 0.95;
                sfx.dash();
                spawnParticles(player.x + player.w / 2, player.y - player.h / 2, '#00f0ff', 8, 110);
            }

            if (player.dashTimer > 0) {
                player.dashTimer -= dt;
                player.x += player.facing * (player.speed * 2.5) * dt;
            } else {
                player.x += moveDir * player.speed * dt;
            }

            // Player jump & gravity
            if ((keys['ArrowUp'] || keys['KeyW']) && player.onGround) {
                player.vy = player.jumpForce;
                player.onGround = false;
                sfx.jump();
            }

            player.vy += player.gravity * dt;
            player.y += player.vy * dt;

            if (player.y >= GROUND_Y) {
                player.y = GROUND_Y;
                player.vy = 0;
                player.onGround = true;
            }

            player.x = Math.max(12, Math.min(GAME_W - player.w - 12, player.x));

            // Player shooting
            if ((keys['Space'] || keys['KeyZ']) && player.shootCooldown <= 0) {
                const buff = player.activeBuff;
                const spread = buff ? buff.spread : 1;
                const fireRate = buff ? buff.fireRate : 0.22;
                let baseDmg = buff ? buff.damage : 14;

                // Double shot reduces per-bolt damage by 5%; Triple shot reduces by 10%
                if (spread === 2) {
                    baseDmg *= 0.95;
                } else if (spread === 3) {
                    baseDmg *= 0.90;
                }
                // Flame Handguns permanent power boosts damage by 1.5x
                if (player.permanentPower && player.permanentPower.id === 'FLAME') {
                    baseDmg *= 1.5;
                }

                const finalDmg = Math.max(1, Math.round(baseDmg));
                const bulletColor = player.permanentPower
                    ? player.permanentPower.bulletColor
                    : (buff ? buff.color : '#00f0ff');

                const gunX = player.facing === 1 ? player.x + player.w + 4 : player.x - 4;
                const gunY = player.y - 22;

                for (let i = 0; i < spread; i++) {
                    const offsetAngle = spread === 1 ? 0 : (i - (spread - 1) / 2) * 0.13;
                    const spd = 540;
                    playerBullets.push({
                        x: gunX,
                        y: gunY,
                        vx: Math.cos(offsetAngle) * spd * player.facing,
                        vy: Math.sin(offsetAngle) * spd,
                        r: player.permanentPower && player.permanentPower.id === 'FLAME' ? 5 : 4,
                        damage: finalDmg,
                        color: bulletColor
                    });
                }

                player.shootCooldown = fireRate;
                sfx.shoot(spread);
            }
        }

        function updateBoss(dt) {
            const cfg = BOSS_STAGES[boss.stageIndex];
            if (boss.hitFlash > 0) boss.hitFlash -= dt;
            if (boss.stunTimer > 0) boss.stunTimer -= dt;
            if (boss.iceTimer > 0) boss.iceTimer -= dt;

            // Calculate boss time dilation from Chrono Slow-Down (50%) or Ice Freeze (50%)
            let bossTimeScale = 1.0;
            if (player.permanentPower && player.permanentPower.id === 'SLOWDOWN') {
                bossTimeScale *= 0.5;
            }
            if (boss.iceTimer > 0) {
                bossTimeScale *= 0.5;
            }
            if (boss.stunTimer > 0) {
                bossTimeScale = 0;
            }

            const bDt = dt * bossTimeScale;
            boss.animTimer += bDt * 7;
            boss.orbitAngle += bDt * 2.4;

            const bossCenter = boss.x + boss.w / 2;
            const targetCenter = player.x + player.w / 2;
            if (!boss.isJumping && boss.chargeTimer <= 0 && bossTimeScale > 0) {
                boss.facing = targetCenter < bossCenter ? -1 : 1;
            }

            // Vacuum Vortex Pull (Stage 7 special)
            if (boss.vortexTimer > 0) {
                boss.vortexTimer -= bDt;
                const pullDir = bossCenter > targetCenter ? 1 : -1;
                player.x = Math.max(12, Math.min(GAME_W - player.w - 12, player.x + pullDir * 115 * dt));
            }

            if (boss.isJumping) {
                boss.x += boss.vx * bDt;
                boss.vy += 860 * bDt;
                boss.y += boss.vy * bDt;
                boss.x = Math.max(24, Math.min(GAME_W - boss.w - 24, boss.x));

                if (boss.y >= GROUND_Y) {
                    boss.y = GROUND_Y;
                    boss.vy = 0;
                    boss.vx = 0;
                    boss.isJumping = false;
                    screenShake = 10;
                    sfx.roar();
                    spawnParticles(bossCenter, GROUND_Y, cfg.accentColor, 18, 160);

                    const dinoDmgScale = cfg.damageMult * 0.85;
                    [-1, 1].forEach(dir => {
                        bossProjectiles.push({
                            type: 'shockwave',
                            x: bossCenter,
                            y: GROUND_Y - 10,
                            vx: dir * (230 + boss.stageIndex * 12),
                            vy: 0,
                            r: 11,
                            damage: 13 * dinoDmgScale,
                            color: cfg.accentColor
                        });
                    });
                }
            } else {
                // Broomstick Charge special (Stage 2) or loose non-chasing patrol
                if (boss.chargeTimer > 0) {
                    boss.chargeTimer -= bDt;
                    boss.x += boss.facing * (cfg.speed * 2.55) * bDt;
                } else {
                    // Loose pacing patrol instead of relentlessly chasing the player
                    boss.patrolTimer -= bDt;
                    if (boss.patrolTimer <= 0) {
                        boss.patrolTimer = 1.4 + Math.random() * 1.4;
                        boss.patrolDir = Math.random() < 0.25 ? 0 : (Math.random() < 0.5 ? -1 : 1);
                    }
                    const patrolSpd = cfg.speed * 0.42;
                    boss.x += boss.patrolDir * patrolSpd * bDt;
                    if (boss.x <= 28) {
                        boss.x = 28;
                        boss.patrolDir = 1;
                    } else if (boss.x >= GAME_W - boss.w - 28) {
                        boss.x = GAME_W - boss.w - 28;
                        boss.patrolDir = -1;
                    }
                }
                boss.x = Math.max(24, Math.min(GAME_W - boss.w - 24, boss.x));
            }

            // Dedicated Gap-Closing Jump Attack every 5 to 10 seconds
            if (!boss.isJumping && boss.chargeTimer <= 0) {
                boss.jumpCooldown -= bDt;
                if (boss.jumpCooldown <= 0) {
                    boss.jumpCooldown = 5.0 + Math.random() * 5.0;
                    boss.isJumping = true;
                    boss.vy = -460;
                    const jumpTime = (2 * 460) / 860;
                    const desiredVx = (targetCenter - bossCenter) / jumpTime;
                    boss.vx = Math.max(-440, Math.min(440, desiredVx));
                    sfx.jump();
                    spawnFloatingText(bossCenter, boss.y - boss.h - 10, '⚡ LEAP SLAM!', cfg.accentColor);
                    spawnParticles(bossCenter, GROUND_Y, '#ffffff', 12, 110);
                }
            }

            // Orbiting porcelain teacups collision (15% reduced damage)
            if (cfg.orbitCups > 0) {
                for (let i = 0; i < cfg.orbitCups; i++) {
                    const ang = boss.orbitAngle + (i * Math.PI * 2) / cfg.orbitCups;
                    const cx = bossCenter + Math.cos(ang) * 76;
                    const cy = (boss.y - boss.h * 0.52) + Math.sin(ang) * 44;
                    if (
                        Math.abs(cx - (player.x + player.w / 2)) < 18 &&
                        Math.abs(cy - (player.y - player.h / 2)) < 22
                    ) {
                        damagePlayer(8 * cfg.damageMult * 0.85);
                    }
                }
            }

            // Boss 5 Standard Attacks + 1 Unique Stage Special
            boss.attackTimer -= bDt;
            if (boss.attackTimer <= 0 && !boss.isJumping) {
                boss.attackTimer = cfg.attackInterval;
                boss.attackCounter++;

                const dinoDmgScale = cfg.damageMult * 0.85;
                const mouthX = boss.facing === -1 ? boss.x + 12 : boss.x + boss.w - 12;
                const mouthY = boss.y - boss.h + 36;
                const dusterX = boss.facing === -1 ? boss.x - 4 : boss.x + boss.w + 4;
                const dusterY = boss.y - 72;

                if (boss.attackCounter % 4 === 0) {
                    triggerStageUniqueSpecial(cfg, bossCenter, mouthX, mouthY, dusterX, dusterY, targetCenter);
                } else {
                    const stdRoll = Math.floor(Math.random() * 5);
                    sfx.roar();

                    if (stdRoll === 0) {
                        // 1. Dust Tornado Stack
                        const count = boss.stageIndex >= 5 ? 3 : 2;
                        for (let i = 0; i < count; i++) {
                            bossProjectiles.push({
                                type: 'tornado',
                                x: dusterX,
                                y: GROUND_Y - 18 - i * 32,
                                vx: boss.facing * (165 + boss.stageIndex * 8),
                                vy: 0,
                                r: 12,
                                damage: 11 * dinoDmgScale,
                                color: '#a855f7'
                            });
                        }
                    } else if (stdRoll === 1) {
                        // 2. Arcing Porcelain Teacup Volley
                        const cups = Math.min(8, 3 + Math.floor(boss.stageIndex / 2));
                        for (let i = 0; i < cups; i++) {
                            bossProjectiles.push({
                                type: 'teacup',
                                x: mouthX,
                                y: mouthY,
                                vx: boss.facing * (135 + i * 34),
                                vy: -220 - i * 28,
                                gravity: 460,
                                r: 8,
                                damage: 10 * dinoDmgScale,
                                color: '#ffffff'
                            });
                        }
                    } else if (stdRoll === 2) {
                        // 3. Curtsy Ground Shockwave
                        const dirs = boss.stageIndex >= 4 ? [-1, 1] : [boss.facing];
                        dirs.forEach(dir => {
                            bossProjectiles.push({
                                type: 'shockwave',
                                x: bossCenter,
                                y: GROUND_Y - 10,
                                vx: dir * (225 + boss.stageIndex * 10),
                                vy: 0,
                                r: 11,
                                damage: 12 * dinoDmgScale,
                                color: cfg.accentColor
                            });
                        });
                    } else if (stdRoll === 3) {
                        // 4. Aimed Feather Duster Orb Spread
                        const spreadCount = boss.stageIndex >= 5 ? 5 : 3;
                        const baseAng = Math.atan2((player.y - player.h / 2) - dusterY, targetCenter - dusterX);
                        for (let i = 0; i < spreadCount; i++) {
                            const ang = baseAng + (i - (spreadCount - 1) / 2) * 0.18;
                            const spd = 210 + boss.stageIndex * 8;
                            bossProjectiles.push({
                                type: 'feather_orb',
                                x: dusterX,
                                y: dusterY,
                                vx: Math.cos(ang) * spd,
                                vy: Math.sin(ang) * spd,
                                r: 8,
                                damage: 10 * dinoDmgScale,
                                color: cfg.accentColor
                            });
                        }
                    } else {
                        // 5. Bouncing Sugar-Cube Bombs
                        for (let i = 0; i < 2; i++) {
                            bossProjectiles.push({
                                type: 'sugar_bomb',
                                x: mouthX,
                                y: mouthY,
                                vx: boss.facing * (150 + i * 55),
                                vy: -180 - i * 30,
                                gravity: 480,
                                bounces: 2,
                                r: 9,
                                damage: 11 * dinoDmgScale,
                                color: '#ffd700'
                            });
                        }
                    }
                }
            }

            // Body contact check (15% reduced dinosaur damage)
            if (
                player.x < boss.x + boss.w - 12 &&
                player.x + player.w > boss.x + 12 &&
                player.y > boss.y - boss.h + 14 &&
                player.y - player.h < boss.y
            ) {
                damagePlayer(10 * cfg.damageMult * 0.85);
            }
        }

        function triggerStageUniqueSpecial(cfg, bossCenter, mouthX, mouthY, dusterX, dusterY, targetCenter) {
            sfx.roar();
            spawnFloatingText(bossCenter, boss.y - boss.h - 14, `★ ${cfg.specialName}! ★`, cfg.accentColor);
            const dinoDmgScale = cfg.damageMult * 0.85; // 15% less dinosaur damage

            switch (cfg.stage) {
                case 1: {
                    // Stage 1 Special: SOAP BUBBLE BARRAGE (floating wavy soap bubbles)
                    for (let i = 0; i < 5; i++) {
                        bossProjectiles.push({
                            type: 'bubble',
                            x: dusterX,
                            y: dusterY - 20 + i * 14,
                            vx: boss.facing * (120 + i * 18),
                            vy: Math.sin(i) * 45,
                            waveOffset: i * 1.3,
                            r: 10,
                            damage: 9 * dinoDmgScale,
                            color: '#80f5ff'
                        });
                    }
                    break;
                }
                case 2: {
                    // Stage 2 Special: BROOMSTICK CHARGE (high-speed dash across arena)
                    boss.chargeTimer = 0.65;
                    screenShake = 7;
                    break;
                }
                case 3: {
                    // Stage 3 Special: BOILING KETTLE GEYSER (erupting steam pillars from ground)
                    for (let i = 0; i < 4; i++) {
                        const gx = Math.max(40, Math.min(GAME_W - 40, targetCenter + (i - 1.5) * 75));
                        bossProjectiles.push({
                            type: 'geyser',
                            x: gx,
                            y: GROUND_Y - 6,
                            vx: 0,
                            vy: -260 - i * 25,
                            gravity: 340,
                            r: 11,
                            damage: 11 * dinoDmgScale,
                            color: '#ffffff'
                        });
                    }
                    break;
                }
                case 4: {
                    // Stage 4 Special: SILVERWARE FAN VOLLEY (wide 6-blade fork/knife fan)
                    for (let i = 0; i < 6; i++) {
                        const ang = (boss.facing === -1 ? Math.PI : 0) + (i - 2.5) * 0.16;
                        bossProjectiles.push({
                            type: 'silverware',
                            x: dusterX,
                            y: dusterY,
                            vx: Math.cos(ang) * 265,
                            vy: Math.sin(ang) * 265,
                            r: 7,
                            damage: 10 * dinoDmgScale,
                            color: '#e2e8f0'
                        });
                    }
                    break;
                }
                case 5: {
                    // Stage 5 Special: CHANDELIER SHATTER (bouncing golden crystal bombs)
                    for (let i = 0; i < 3; i++) {
                        bossProjectiles.push({
                            type: 'sugar_bomb',
                            x: mouthX,
                            y: mouthY - 12,
                            vx: boss.facing * (145 + i * 65),
                            vy: -170 - i * 35,
                            gravity: 500,
                            bounces: 3,
                            r: 10,
                            damage: 11 * dinoDmgScale,
                            color: '#ffd700'
                        });
                    }
                    break;
                }
                case 6: {
                    // Stage 6 Special: POLISH MIRROR BEAM (fast horizontal laser stream)
                    screenShake = 8;
                    for (let i = 0; i < 7; i++) {
                        bossProjectiles.push({
                            type: 'mirror_beam',
                            x: mouthX + boss.facing * i * 22,
                            y: GROUND_Y - 22,
                            vx: boss.facing * 360,
                            vy: 0,
                            r: 8,
                            damage: 11 * dinoDmgScale,
                            color: '#38bdf8'
                        });
                    }
                    break;
                }
                case 7: {
                    // Stage 7 Special: VACUUM VORTEX PULL (pulls hero inward while launching orbs)
                    boss.vortexTimer = 2.0;
                    for (let i = 0; i < 4; i++) {
                        bossProjectiles.push({
                            type: 'tornado',
                            x: bossCenter,
                            y: GROUND_Y - 22 - i * 34,
                            vx: boss.facing * 195,
                            vy: 0,
                            r: 11,
                            damage: 11 * dinoDmgScale,
                            color: '#a855f7'
                        });
                    }
                    break;
                }
                case 8: {
                    // Stage 8 Special: SOLAR DUSTER NOVA (360-degree radial ring burst)
                    screenShake = 9;
                    const ringCount = 10;
                    for (let i = 0; i < ringCount; i++) {
                        const ang = (i * Math.PI * 2) / ringCount;
                        bossProjectiles.push({
                            type: 'feather_orb',
                            x: bossCenter,
                            y: boss.y - boss.h / 2,
                            vx: Math.cos(ang) * 200,
                            vy: Math.sin(ang) * 200,
                            r: 8,
                            damage: 11 * dinoDmgScale,
                            color: '#ffd700'
                        });
                    }
                    break;
                }
                case 9: {
                    // Stage 9 Special: GRANDFATHER CLOCK SPIRAL (accelerating time-gear projectiles)
                    for (let i = 0; i < 6; i++) {
                        const ang = (boss.facing === -1 ? Math.PI : 0) + (i - 2.5) * 0.24;
                        bossProjectiles.push({
                            type: 'clock_gear',
                            x: dusterX,
                            y: dusterY,
                            vx: Math.cos(ang) * 140,
                            vy: Math.sin(ang) * 140,
                            accel: 160,
                            r: 9,
                            damage: 12 * dinoDmgScale,
                            color: '#39ff14'
                        });
                    }
                    break;
                }
                case 10: {
                    // Stage 10 Special: SPAWN 3 LITTLE MAID RAPTORS (100 HP each, 5 melee damage every 1.5s)
                    screenShake = 10;
                    for (let i = 0; i < 3; i++) {
                        if (miniRaptors.length >= 6) break;
                        const spawnX = Math.max(30, Math.min(GAME_W - 60, bossCenter + (i - 1) * 44));
                        miniRaptors.push({
                            x: spawnX,
                            y: GROUND_Y,
                            w: 28,
                            h: 34,
                            hp: 100,
                            maxHp: 100,
                            speed: 145 + i * 12,
                            facing: boss.facing,
                            meleeCooldown: 0.4 + i * 0.2,
                            hitFlash: 0,
                            animTimer: i * 1.5
                        });
                        spawnParticles(spawnX + 14, GROUND_Y - 16, '#ff2a4b', 10, 110);
                    }
                    break;
                }
            }
        }

        // ============================================================================
        // 10. PROJECTILE & EFFECT UPDATER
        // ============================================================================
        function updateProjectilesAndEffects(dt) {
            const pScale = (player.permanentPower && player.permanentPower.id === 'SLOWDOWN') ? 0.5 : 1.0;
            const pDt = dt * pScale;

            // Update player bullets
            for (let i = playerBullets.length - 1; i >= 0; i--) {
                const b = playerBullets[i];
                b.x += b.vx * dt;
                b.y += b.vy * dt;

                if (b.x < -20 || b.x > GAME_W + 20 || b.y < -20 || b.y > GROUND_Y + 10) {
                    playerBullets.splice(i, 1);
                    continue;
                }

                // Check hit against Mini Maid Raptors first
                let hitRaptor = false;
                for (let rIdx = miniRaptors.length - 1; rIdx >= 0; rIdx--) {
                    const mr = miniRaptors[rIdx];
                    if (
                        b.x + b.r > mr.x &&
                        b.x - b.r < mr.x + mr.w &&
                        b.y + b.r > mr.y - mr.h &&
                        b.y - b.r < mr.y
                    ) {
                        mr.hp -= b.damage;
                        mr.hitFlash = 0.08;
                        stats.score += b.damage * 3;
                        sfx.bossHit();
                        spawnParticles(b.x, b.y, b.color, 4, 70);
                        spawnFloatingText(b.x, b.y - 8, `-${b.damage}`, '#ffd700');
                        playerBullets.splice(i, 1);
                        hitRaptor = true;

                        if (mr.hp <= 0) {
                            spawnParticles(mr.x + mr.w / 2, mr.y - mr.h / 2, '#ff2a4b', 14, 130);
                            spawnFloatingText(mr.x + mr.w / 2, mr.y - mr.h - 6, 'RAPTOR DUSTED!', '#39ff14');
                            miniRaptors.splice(rIdx, 1);
                        }
                        break;
                    }
                }
                if (hitRaptor) continue;

                // Boss hit check
                if (
                    b.x + b.r > boss.x + 8 &&
                    b.x - b.r < boss.x + boss.w - 8 &&
                    b.y + b.r > boss.y - boss.h + 10 &&
                    b.y - b.r < boss.y
                ) {
                    boss.hp -= b.damage;
                    boss.hitFlash = 0.08;
                    stats.score += b.damage * 4;
                    sfx.bossHit();
                    spawnParticles(b.x, b.y, b.color, 5, 80);
                    spawnFloatingText(b.x, b.y - 8, `-${b.damage}`, '#ffd700');
                    playerBullets.splice(i, 1);

                    if (boss.hp <= 0) {
                        boss.hp = 0;
                        if (boss.stageIndex + 1 < BOSS_STAGES.length) {
                            triggerStageTransition(boss.stageIndex + 1);
                        } else {
                            triggerEndGame(true);
                        }
                        return;
                    }
                }
            }

            // Update Mini Maid Raptors (Stage 10 Special: 100 HP, 5 melee damage every 1.5s)
            for (let i = miniRaptors.length - 1; i >= 0; i--) {
                const mr = miniRaptors[i];
                if (mr.hitFlash > 0) mr.hitFlash -= dt;
                if (mr.meleeCooldown > 0) mr.meleeCooldown -= pDt;
                mr.animTimer += pDt * 10;

                const targetX = player.x + player.w / 2;
                const mrCenter = mr.x + mr.w / 2;
                const dist = targetX - mrCenter;
                mr.facing = dist < 0 ? -1 : 1;

                if (Math.abs(dist) > 16) {
                    mr.x += mr.facing * mr.speed * pDt;
                }

                // Melee attack check: 5 melee damage every 1.5 seconds
                if (
                    player.x < mr.x + mr.w &&
                    player.x + player.w > mr.x &&
                    player.y > mr.y - mr.h &&
                    player.y - player.h < mr.y &&
                    mr.meleeCooldown <= 0
                ) {
                    mr.meleeCooldown = 1.5;
                    damagePlayer(5);
                }
            }

            // Update boss projectiles (affected by 1-in-50 Chrono Slow-Down!)
            for (let i = bossProjectiles.length - 1; i >= 0; i--) {
                const p = bossProjectiles[i];

                if (p.type === 'bubble') {
                    p.waveOffset += pDt * 5;
                    p.vy = Math.sin(p.waveOffset) * 65;
                } else if (p.type === 'clock_gear') {
                    const ang = Math.atan2(p.vy, p.vx);
                    p.vx += Math.cos(ang) * p.accel * pDt;
                    p.vy += Math.sin(ang) * p.accel * pDt;
                }

                if (p.gravity) {
                    p.vy += p.gravity * pDt;
                }

                p.x += p.vx * pDt;
                p.y += p.vy * pDt;

                // Floor interactions
                if (p.y >= GROUND_Y - p.r) {
                    if (p.type === 'sugar_bomb' && p.bounces > 0) {
                        p.y = GROUND_Y - p.r;
                        p.vy = -Math.abs(p.vy) * 0.78;
                        p.bounces--;
                        spawnParticles(p.x, GROUND_Y, '#ffd700', 5, 70);
                    } else if (p.type === 'teacup' || p.type === 'sugar_bomb' || p.type === 'geyser') {
                        spawnParticles(p.x, GROUND_Y, p.color, 6, 85);
                        bossProjectiles.splice(i, 1);
                        continue;
                    }
                }

                if (p.x < -40 || p.x > GAME_W + 40 || p.y < -60 || p.y > GAME_H + 40) {
                    bossProjectiles.splice(i, 1);
                    continue;
                }

                // Player collision
                const px = player.x + player.w / 2;
                const py = player.y - player.h / 2;
                if (Math.hypot(p.x - px, p.y - py) < p.r + 12) {
                    damagePlayer(p.damage);
                    spawnParticles(p.x, p.y, p.color, 6, 80);
                    bossProjectiles.splice(i, 1);
                }
            }

            // Update particles
            for (let i = particles.length - 1; i >= 0; i--) {
                const pt = particles[i];
                pt.x += pt.vx * dt;
                pt.y += pt.vy * dt;
                pt.vy += 260 * dt;
                pt.life -= dt;
                if (pt.life <= 0) particles.splice(i, 1);
            }

            // Update floating texts
            for (let i = floatingTexts.length - 1; i >= 0; i--) {
                const ft = floatingTexts[i];
                ft.y -= 28 * dt;
                ft.life -= dt;
                if (ft.life <= 0) floatingTexts.splice(i, 1);
            }
        }

        // ============================================================================
        // 11. PIXEL-ART RENDERING ENGINE
        // ============================================================================
        function drawArenaBackground() {
            // 1. Deep Retro Synthwave Twilight Sky (Darkened lower horizon for crisp foreground contrast)
            const skyGrad = ctx.createLinearGradient(0, 0, 0, GROUND_Y);
            skyGrad.addColorStop(0, '#070214');
            skyGrad.addColorStop(0.42, '#1d0742');
            skyGrad.addColorStop(0.75, '#470f4a');
            skyGrad.addColorStop(1, '#260b36');
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, GAME_W, GROUND_Y);

            // 2. Twinkling Retro Pixel Stars
            const stars = [
                [45, 32], [110, 58], [185, 24], [250, 48], [320, 20],
                [480, 28], [550, 52], [630, 22], [715, 44], [765, 26],
                [75, 92], [295, 84], [515, 88], [685, 95]
            ];
            stars.forEach(([sx, sy], idx) => {
                ctx.fillStyle = idx % 2 === 0 ? 'rgba(255,255,255,0.75)' : 'rgba(0,240,255,0.65)';
                ctx.fillRect(sx, sy, 3, 3);
            });

            // 3. Striped Outrun Sun (Positioned higher in the sky out of the combat lane)
            const sunX = GAME_W / 2;
            const sunY = 175;
            const sunR = 72;
            const sunGrad = ctx.createLinearGradient(sunX, sunY - sunR, sunX, sunY + sunR);
            sunGrad.addColorStop(0, '#d9b836');
            sunGrad.addColorStop(0.5, '#c95324');
            sunGrad.addColorStop(1, '#9e1955');
            ctx.fillStyle = sunGrad;
            ctx.beginPath();
            ctx.arc(sunX, sunY, sunR, 0, Math.PI * 2);
            ctx.fill();

            // Horizontal Synthwave Scan-Bands across Lower Sun
            ctx.fillStyle = '#26093b';
            ctx.fillRect(sunX - sunR, sunY + 14, sunR * 2, 4);
            ctx.fillRect(sunX - sunR, sunY + 26, sunR * 2, 6);
            ctx.fillRect(sunX - sunR, sunY + 40, sunR * 2, 8);
            ctx.fillRect(sunX - sunR, sunY + 54, sunR * 2, 10);

            // 4. Far Background City Skyline Silhouettes
            const farBuildings = [
                { x: 0, w: 68, h: 195, col: '#120a2b' },
                { x: 64, w: 82, h: 245, col: '#170d36' },
                { x: 142, w: 60, h: 175, col: '#100826' },
                { x: 198, w: 94, h: 270, col: '#1a0e3d' },
                { x: 288, w: 66, h: 185, col: '#120a2b' },
                { x: 455, w: 72, h: 205, col: '#120a2b' },
                { x: 522, w: 96, h: 265, col: '#1a0e3d' },
                { x: 614, w: 64, h: 180, col: '#100826' },
                { x: 674, w: 80, h: 235, col: '#170d36' },
                { x: 750, w: 54, h: 190, col: '#120a2b' }
            ];
            farBuildings.forEach(b => {
                ctx.fillStyle = b.col;
                ctx.fillRect(b.x, GROUND_Y - b.h, b.w, b.h);
                // Radio Antenna Spire
                ctx.fillStyle = '#8c1d52';
                ctx.fillRect(b.x + Math.floor(b.w / 2), GROUND_Y - b.h - 14, 2, 14);
                ctx.fillStyle = '#0c7c8c';
                ctx.fillRect(b.x + Math.floor(b.w / 2) - 1, GROUND_Y - b.h - 17, 4, 4);
            });

            // 5. Foreground Muted Neon Skyscrapers with Softened Windows
            const frontBuildings = [
                { x: 12, w: 74, h: 155, body: '#0d102b', trim: '#14889c', win: '#7a6829' },
                { x: 96, w: 88, h: 210, body: '#180a2e', trim: '#991f56', win: '#17788a' },
                { x: 194, w: 70, h: 135, body: '#0b1c2e', trim: '#238c14', win: '#6e7385' },
                { x: 274, w: 82, h: 175, body: '#1f0b2b', trim: '#998217', win: '#8c4261' },
                { x: 446, w: 80, h: 170, body: '#0d102b', trim: '#991f56', win: '#7a6829' },
                { x: 536, w: 92, h: 215, body: '#0b1e29', trim: '#14889c', win: '#227a14' },
                { x: 638, w: 68, h: 142, body: '#1d0b2b', trim: '#998217', win: '#17788a' },
                { x: 716, w: 76, h: 185, body: '#100d2e', trim: '#238c14', win: '#8c2252' }
            ];

            frontBuildings.forEach((b, bIdx) => {
                const topY = GROUND_Y - b.h;
                ctx.fillStyle = b.body;
                ctx.fillRect(b.x, topY, b.w, b.h);

                // Subtle Rooftop Trim & Side Edge Highlight
                ctx.fillStyle = b.trim;
                ctx.fillRect(b.x, topY, b.w, 3);
                ctx.fillRect(b.x, topY, 2, b.h);

                // Dimmed Pixel Window Grid (Keeps upper skyline colorful without distracting near the ground)
                for (let wy = topY + 14; wy < GROUND_Y - 58; wy += 16) {
                    for (let wx = b.x + 10; wx < b.x + b.w - 14; wx += 14) {
                        const pattern = (wx + wy + bIdx * 7) % 6;
                        if (pattern === 1 || pattern === 3) {
                            ctx.fillStyle = b.win;
                            ctx.fillRect(wx, wy, 6, 8);
                        } else {
                            ctx.fillStyle = '#070514';
                            ctx.fillRect(wx, wy, 6, 8);
                        }
                    }
                }
            });

            // 6. Darkened Combat-Zone Atmospheric Band (Makes Hero & Projectiles Pop Clearly)
            const combatFog = ctx.createLinearGradient(0, GROUND_Y - 110, 0, GROUND_Y);
            combatFog.addColorStop(0, 'rgba(7, 4, 18, 0.0)');
            combatFog.addColorStop(1, 'rgba(7, 4, 18, 0.68)');
            ctx.fillStyle = combatFog;
            ctx.fillRect(0, GROUND_Y - 110, GAME_W, 110);

            // 7. Retro Neon Streetlamps Along the Sidewalk
            for (let lx = 55; lx < GAME_W; lx += 170) {
                ctx.fillStyle = '#334155';
                ctx.fillRect(lx, GROUND_Y - 46, 4, 46);
                ctx.fillStyle = '#189ab4';
                ctx.fillRect(lx - 4, GROUND_Y - 50, 12, 4);
            }

            // 8. Neon-Trimmed Retro City Boulevard & Asphalt Street
            ctx.fillStyle = '#0d091c';
            ctx.fillRect(0, GROUND_Y, GAME_W, GAME_H - GROUND_Y);

            // Glowing Sidewalk Curb
            ctx.fillStyle = '#00f0ff';
            ctx.fillRect(0, GROUND_Y, GAME_W, 3);
            ctx.fillStyle = '#ff2a85';
            ctx.fillRect(0, GROUND_Y + 3, GAME_W, 2);

            // Muted Gold Arcade Street Lane Lines
            const dashW = 36;
            for (let x = 12; x < GAME_W; x += dashW * 2) {
                ctx.fillStyle = '#856e14';
                ctx.fillRect(x, GROUND_Y + 26, dashW, 4);
            }
        }

        function drawDinoMaid() {
            const cfg = BOSS_STAGES[boss.stageIndex];
            const cx = boss.x + boss.w / 2;

            // Boss Stage Aura
            if (cfg.auraColor) {
                ctx.fillStyle = cfg.auraColor;
                ctx.beginPath();
                ctx.ellipse(cx, boss.y - boss.h * 0.48, 72, 78, 0, 0, Math.PI * 2);
                ctx.fill();
            }

            // Orbiting Porcelain Teacups
            if (cfg.orbitCups > 0) {
                for (let i = 0; i < cfg.orbitCups; i++) {
                    const ang = boss.orbitAngle + (i * Math.PI * 2) / cfg.orbitCups;
                    const ox = cx + Math.cos(ang) * 76;
                    const oy = (boss.y - boss.h * 0.52) + Math.sin(ang) * 44;
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(ox - 6, oy - 4, 12, 8);
                    ctx.fillStyle = cfg.accentColor;
                    ctx.fillRect(ox - 5, oy - 2, 10, 2);
                }
            }

            ctx.save();
            ctx.translate(cx, boss.y);
            if (boss.facing === 1) ctx.scale(-1, 1);

            const scaleMain = boss.hitFlash > 0
                ? '#ffffff'
                : (boss.stunTimer > 0 ? '#ffe600' : (boss.iceTimer > 0 ? '#7dd3fc' : cfg.scaleColor));
            const scaleDark = boss.hitFlash > 0 ? '#e2e8f0' : cfg.scaleDark;
            const step = boss.isJumping ? 0 : Math.sin(boss.animTimer) * 4;
            const jawOpen = Math.abs(Math.sin(boss.animTimer * 0.7)) * 6;

            // 1. Segmented Wagging T-Rex Tail + Dorsal Spikes
            ctx.fillStyle = scaleMain;
            ctx.fillRect(28, -54 + step * 0.4, 24, 20);
            ctx.fillRect(50, -48 - step * 0.5, 22, 14);
            ctx.fillRect(70, -43 + step * 0.6, 18, 9);
            // Dorsal Tail Spikes
            ctx.fillStyle = cfg.accentColor;
            ctx.fillRect(36, -61 + step * 0.4, 7, 7);
            ctx.fillRect(56, -55 - step * 0.5, 6, 7);
            ctx.fillRect(74, -49 + step * 0.6, 5, 6);

            // 2. Muscular Clawed Legs
            ctx.fillStyle = scaleDark;
            ctx.fillRect(-24, -26 + step, 16, 26 - step);
            ctx.fillRect(6, -26 - step, 16, 26 + step);
            // White Claws
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-28, -5, 5, 5);
            ctx.fillRect(2, -5, 5, 5);

            // 3. Victorian Maid Dress & White Frilly Apron
            ctx.fillStyle = '#14101f';
            ctx.beginPath();
            ctx.moveTo(-34, -22);
            ctx.lineTo(34, -22);
            ctx.lineTo(24, -76);
            ctx.lineTo(-24, -76);
            ctx.closePath();
            ctx.fill();

            // White Apron & Lace Hem
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-26, -68, 28, 42);
            for (let lx = -32; lx <= 26; lx += 10) {
                ctx.fillRect(lx, -24, 6, 5);
            }
            // Pink/Accent Bow
            ctx.fillStyle = cfg.accentColor;
            ctx.fillRect(-16, -62, 14, 8);

            // 4. Tiny Stubby T-Rex Arms + Feather Duster
            ctx.fillStyle = scaleMain;
            ctx.fillRect(-34, -68, 14, 8);
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-37, -66, 4, 4);
            // Feather Duster Handle & Plume
            ctx.fillStyle = '#d97706';
            ctx.fillRect(-48, -74, 16, 4);
            ctx.fillStyle = cfg.accentColor;
            ctx.beginPath();
            ctx.arc(-52, -72, 11, 0, Math.PI * 2);
            ctx.fill();

            // 5. Massive T-Rex Head, Snout, Hinged Jaw & Sharp Teeth
            ctx.fillStyle = scaleMain;
            // Upper Cranium & Snout
            ctx.fillRect(-48, -124, 58, 30);
            // Lower Hinged Jaw
            ctx.fillRect(-44, -93 + jawOpen, 46, 11);

            // Upper & Lower Fangs
            ctx.fillStyle = '#ffffff';
            for (let tx = -44; tx <= -14; tx += 10) {
                ctx.fillRect(tx, -94, 4, 5);
                ctx.fillRect(tx + 3, -98 + jawOpen, 4, 5);
            }

            // Fierce Eye & Nostril
            ctx.fillStyle = '#ffd700';
            ctx.fillRect(-16, -116, 10, 10);
            ctx.fillStyle = '#090512';
            ctx.fillRect(-14, -114, 5, 6);
            ctx.fillStyle = scaleDark;
            ctx.fillRect(-42, -116, 4, 4);

            // 6. Frilly Maid Headband & Optional Royal Crown
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-26, -132, 34, 8);
            ctx.fillStyle = cfg.accentColor;
            ctx.fillRect(-20, -135, 22, 4);

            if (cfg.crown) {
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(-16, -145, 20, 10);
                ctx.fillRect(-16, -149, 4, 4);
                ctx.fillRect(-8, -150, 4, 5);
                ctx.fillRect(0, -149, 4, 4);
            }

            ctx.restore();
        }

        function drawPlayer() {
            if (player.invulnTimer > 0 && Math.floor(player.invulnTimer * 18) % 2 === 0) return;

            const cx = player.x + player.w / 2;

            // Crisp Ground Drop Shadow & Subtle Hero Spotlight Glow
            ctx.fillStyle = 'rgba(0, 0, 0, 0.65)';
            ctx.beginPath();
            ctx.ellipse(cx, GROUND_Y + 2, 16, 4, 0, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = 'rgba(0, 240, 255, 0.14)';
            ctx.beginPath();
            ctx.ellipse(cx, player.y - player.h / 2, 22, 26, 0, 0, Math.PI * 2);
            ctx.fill();

            ctx.save();
            ctx.translate(cx, player.y);
            if (player.facing === -1) ctx.scale(-1, 1);

            const legSwing = player.onGround ? Math.sin(player.animFrame) * 4 : 3;

            // High-Contrast White/Dark Silhouette Backing Outline
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-11, -44, 22, 33);

            // Boots
            ctx.fillStyle = '#1e293b';
            ctx.fillRect(-9, -12 + legSwing, 7, 12 - legSwing);
            ctx.fillRect(2, -12 - legSwing, 7, 12 + legSwing);

            // Armor Torso
            ctx.fillStyle = player.permanentPower ? player.permanentPower.bulletColor : '#00f0ff';
            ctx.fillRect(-10, -28, 20, 16);
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(-6, -25, 12, 8);

            // Helmet & Visor
            ctx.fillStyle = '#38bdf8';
            ctx.fillRect(-9, -42, 18, 14);
            ctx.fillStyle = '#ffd700';
            ctx.fillRect(0, -38, 9, 5);

            // Blaster / Flame Handguns
            ctx.fillStyle = player.permanentPower && player.permanentPower.id === 'FLAME' ? '#ff6b1a' : '#ffffff';
            ctx.fillRect(8, -25, 13, 6);

            ctx.restore();
        }

        function drawProjectilesAndEffects() {
            // Mini Maid Raptors (Stage 10 Special)
            miniRaptors.forEach(mr => {
                ctx.save();
                const cx = mr.x + mr.w / 2;
                ctx.translate(cx, mr.y);
                if (mr.facing === 1) ctx.scale(-1, 1);

                const bodyCol = mr.hitFlash > 0 ? '#ffffff' : '#ff2a4b';
                const step = Math.sin(mr.animTimer) * 2;

                // Mini Tail & Legs
                ctx.fillStyle = bodyCol;
                ctx.fillRect(10, -16, 12, 6);
                ctx.fillRect(-7, -10 + step, 5, 10 - step);
                ctx.fillRect(2, -10 - step, 5, 10 + step);

                // Mini Maid Dress & Apron
                ctx.fillStyle = '#14101f';
                ctx.fillRect(-10, -22, 20, 13);
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(-9, -20, 9, 10);

                // Mini Raptor Head & Headband
                ctx.fillStyle = bodyCol;
                ctx.fillRect(-15, -33, 18, 11);
                ctx.fillStyle = '#ffd700';
                ctx.fillRect(-6, -30, 4, 4);
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(-8, -36, 10, 3);

                ctx.restore();

                // Mini HP Bar above raptor
                const hpRatio = Math.max(0, mr.hp / mr.maxHp);
                ctx.fillStyle = '#090512';
                ctx.fillRect(mr.x, mr.y - mr.h - 10, mr.w, 4);
                ctx.fillStyle = '#39ff14';
                ctx.fillRect(mr.x, mr.y - mr.h - 10, mr.w * hpRatio, 4);
            });

            // Player bullets
            playerBullets.forEach(b => {
                ctx.fillStyle = b.color;
                ctx.beginPath();
                ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(b.x - 2, b.y - 2, 4, 4);
            });

            // Boss projectiles
            bossProjectiles.forEach(p => {
                ctx.save();
                ctx.translate(p.x, p.y);
                const t = performance.now() * 0.012;

                if (p.type === 'tornado') {
                    // Swirling Layered Dust Cyclone
                    for (let layer = 0; layer < 4; layer++) {
                        const w = 24 - layer * 4;
                        const sway = Math.sin(t * 2.2 + layer * 1.4) * 5;
                        ctx.fillStyle = layer % 2 === 0 ? '#e9d5ff' : '#a855f7';
                        ctx.fillRect(-w / 2 + sway, -12 + layer * 6, w, 4);
                    }
                    // Floating dust specks around cyclone
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(Math.cos(t * 3) * 12, -8, 3, 3);
                    ctx.fillRect(Math.sin(t * 2.5) * -12, 4, 3, 3);

                } else if (p.type === 'teacup') {
                    // Spinning Porcelain Teacup with Handle & Rim
                    ctx.rotate(t * 1.6);
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(-7, -5, 14, 9);
                    ctx.fillRect(-5, 4, 10, 3);
                    // Handle
                    ctx.strokeStyle = '#ffffff';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(6, -3, 4, 5);
                    // Pink/Gold stripe
                    ctx.fillStyle = '#ff2a85';
                    ctx.fillRect(-7, -2, 14, 2);

                } else if (p.type === 'shockwave') {
                    // Jagged Crescent Ground Shockwave
                    const dir = p.vx >= 0 ? 1 : -1;
                    ctx.scale(dir, 1);
                    ctx.fillStyle = p.color;
                    ctx.beginPath();
                    ctx.moveTo(-12, 10);
                    ctx.lineTo(14, 10);
                    ctx.lineTo(6, -12);
                    ctx.lineTo(-4, -2);
                    ctx.lineTo(-10, -8);
                    ctx.closePath();
                    ctx.fill();
                    // Inner bright energy core
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(-4, 2, 12, 5);

                } else if (p.type === 'feather_orb') {
                    // Glowing Feather-Plume Energy Orb with Tail
                    const ang = Math.atan2(p.vy, p.vx);
                    ctx.rotate(ang);
                    ctx.fillStyle = p.color;
                    ctx.beginPath();
                    ctx.ellipse(0, 0, p.r + 4, p.r, 0, 0, Math.PI * 2);
                    ctx.fill();
                    // Wispy feather trail
                    ctx.fillRect(-p.r - 9, -3, 8, 2);
                    ctx.fillRect(-p.r - 7, 1, 6, 2);
                    // White hot center
                    ctx.fillStyle = '#ffffff';
                    ctx.beginPath();
                    ctx.arc(2, 0, p.r * 0.45, 0, Math.PI * 2);
                    ctx.fill();

                } else if (p.type === 'bubble') {
                    // Iridescent Soap Bubble with Specular Shine
                    ctx.fillStyle = 'rgba(128, 245, 255, 0.25)';
                    ctx.beginPath();
                    ctx.arc(0, 0, p.r, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.strokeStyle = '#80f5ff';
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    // Glossy highlight
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(-p.r * 0.45, -p.r * 0.45, 4, 4);

                } else if (p.type === 'geyser') {
                    // Erupting Boiling Steam Jet
                    ctx.fillStyle = 'rgba(255, 255, 255, 0.85)';
                    ctx.beginPath();
                    ctx.arc(0, 0, p.r, 0, Math.PI * 2);
                    ctx.arc(-5, 6, p.r * 0.7, 0, Math.PI * 2);
                    ctx.arc(5, 6, p.r * 0.7, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.fillStyle = '#7dd3fc';
                    ctx.fillRect(-4, -3, 8, 6);

                } else if (p.type === 'silverware') {
                    // Gleaming Silver Knife / Fork Blade
                    const ang = Math.atan2(p.vy, p.vx);
                    ctx.rotate(ang);
                    // Handle
                    ctx.fillStyle = '#94a3b8';
                    ctx.fillRect(-12, -2, 8, 4);
                    // Sharp Blade
                    ctx.fillStyle = '#ffffff';
                    ctx.beginPath();
                    ctx.moveTo(-4, -3);
                    ctx.lineTo(13, 0);
                    ctx.lineTo(-4, 3);
                    ctx.closePath();
                    ctx.fill();

                } else if (p.type === 'sugar_bomb') {
                    // Faceted Golden Chandelier Crystal Bomb
                    ctx.rotate(t * 2.0);
                    ctx.fillStyle = '#ffd700';
                    ctx.beginPath();
                    ctx.moveTo(0, -p.r - 2);
                    ctx.lineTo(p.r + 2, 0);
                    ctx.lineTo(0, p.r + 2);
                    ctx.lineTo(-p.r - 2, 0);
                    ctx.closePath();
                    ctx.fill();
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(-3, -3, 6, 6);

                } else if (p.type === 'mirror_beam') {
                    // Piercing Cyan Laser Bolt with Glowing Core
                    ctx.fillStyle = 'rgba(56, 189, 248, 0.45)';
                    ctx.fillRect(-p.r - 6, -p.r, (p.r + 6) * 2, p.r * 2);
                    ctx.fillStyle = '#38bdf8';
                    ctx.fillRect(-p.r - 4, -p.r * 0.65, (p.r + 4) * 2, p.r * 1.3);
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(-p.r - 2, -2, (p.r + 2) * 2, 4);

                } else if (p.type === 'clock_gear') {
                    // Spinning Neon Clockwork Cog
                    ctx.rotate(t * 2.5);
                    ctx.fillStyle = '#39ff14';
                    for (let tooth = 0; tooth < 4; tooth++) {
                        ctx.rotate(Math.PI / 4);
                        ctx.fillRect(-p.r - 2, -2, (p.r + 2) * 2, 4);
                    }
                    ctx.beginPath();
                    ctx.arc(0, 0, p.r * 0.75, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.fillStyle = '#090612';
                    ctx.beginPath();
                    ctx.arc(0, 0, p.r * 0.35, 0, Math.PI * 2);
                    ctx.fill();

                } else {
                    ctx.fillStyle = p.color;
                    ctx.beginPath();
                    ctx.arc(0, 0, p.r, 0, Math.PI * 2);
                    ctx.fill();
                }

                ctx.restore();
            });

            // Particles
            particles.forEach(pt => {
                ctx.fillStyle = pt.color;
                ctx.fillRect(pt.x, pt.y, pt.size, pt.size);
            });

            // Floating Combat Texts
            ctx.font = '10px "Press Start 2P", monospace';
            ctx.textAlign = 'center';
            floatingTexts.forEach(ft => {
                ctx.fillStyle = '#000000';
                ctx.fillText(ft.text, ft.x + 1, ft.y + 1);
                ctx.fillStyle = ft.color;
                ctx.fillText(ft.text, ft.x, ft.y);
            });

            // Stage Transition Banner
            if (stageTransitionTimer > 0 && stageBannerData) {
                ctx.fillStyle = 'rgba(9, 6, 18, 0.82)';
                ctx.fillRect(140, 145, 520, 110);
                ctx.strokeStyle = '#ffd700';
                ctx.lineWidth = 3;
                ctx.strokeRect(140, 145, 520, 110);

                ctx.textAlign = 'center';
                ctx.fillStyle = '#ffd700';
                ctx.font = '18px "Press Start 2P", monospace';
                ctx.fillText(stageBannerData.title, GAME_W / 2, 182);

                ctx.fillStyle = '#ff2a85';
                ctx.font = '12px "Press Start 2P", monospace';
                ctx.fillText(stageBannerData.subtitle, GAME_W / 2, 210);

                ctx.fillStyle = '#39ff14';
                ctx.font = '10px "Press Start 2P", monospace';
                ctx.fillText(stageBannerData.bonus, GAME_W / 2, 236);
            }
        }

        function updateHUD() {
            const cfg = BOSS_STAGES[boss.stageIndex];
            hudPlayerHpText.textContent = `${player.hp}/${player.maxHp}`;
            hudPlayerHpBar.style.width = `${Math.max(0, (player.hp / player.maxHp) * 100)}%`;

            if (player.activeBuff) {
                hudBuffTag.textContent = `WEAPON: ${player.activeBuff.name} (${player.activeBuff.timer.toFixed(1)}s)`;
            } else {
                hudBuffTag.textContent = 'WEAPON: STANDARD';
            }

            if (player.permanentPower) {
                hudPowerTag.textContent = `POWER: ${player.permanentPower.label}`;
                hudPowerTag.className = 'mt-0.5 text-[7px] sm:text-[8px] text-yellow-300 truncate';
            } else {
                hudPowerTag.textContent = 'POWER: NONE (<10% CHANCE)';
                hudPowerTag.className = 'mt-0.5 text-[7px] sm:text-[8px] text-pink-300 truncate';
            }

            hudKrillionTimer.textContent = `${Math.max(0, krillionTimer).toFixed(1)}s`;
            hudScore.textContent = `SCORE: ${stats.score}`;

            hudStageBadge.textContent = `STG ${cfg.stage}/10`;
            hudBossName.textContent = cfg.name;
            hudBossHpBar.style.width = `${Math.max(0, (boss.hp / boss.maxHp) * 100)}%`;
            hudBossHpText.textContent = `HP: ${Math.max(0, Math.ceil(boss.hp))}/${boss.maxHp}`;
        }

        // ============================================================================
        // 12. MAIN GAME LOOP
        // ============================================================================
        function gameLoop(timestamp) {
            if (!lastTimestamp) lastTimestamp = timestamp;
            const dt = Math.min(0.05, (timestamp - lastTimestamp) / 1000);
            lastTimestamp = timestamp;

            if (gameState === 'PLAYING') {
                stats.survivalTime += dt;
                if (stageTransitionTimer > 0) stageTransitionTimer -= dt;

                krillionTimer -= dt;
                if (krillionTimer <= 0) {
                    triggerKrillionQuiz();
                } else {
                    updatePlayer(dt);
                    updateBoss(dt);
                    updateProjectilesAndEffects(dt);
                    updateHUD();
                }
            } else if (gameState === 'QUIZ') {
                if (!quizInputArea.classList.contains('hidden')) {
                    quizCountdown -= dt;
                    quizTimerEl.textContent = `${Math.max(0, quizCountdown).toFixed(1)}s`;
                    if (quizCountdown <= 0) {
                        submitTriviaAnswer();
                    }
                }
            }

            // Render Frame
            ctx.save();
            if (screenShake > 0) {
                const sx = (Math.random() - 0.5) * screenShake;
                const sy = (Math.random() - 0.5) * screenShake;
                ctx.translate(sx, sy);
                screenShake = Math.max(0, screenShake - 35 * dt);
            }

            drawArenaBackground();
            drawDinoMaid();
            drawPlayer();
            drawProjectilesAndEffects();
            ctx.restore();

            requestAnimationFrame(gameLoop);
        }

        window.onload = function () {
            resizeCanvas();
            requestAnimationFrame(gameLoop);
        };
    </script>
</body>
</html>