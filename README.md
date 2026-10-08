<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>木材加工　シミュレーター：立木から丸太・板の切り出しと木目観察</title>

    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Three.js & OrbitControls for 3D Viewer -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

    <!-- Google Fonts: Noto Sans JP -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700;900&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'Noto Sans JP', sans-serif;
            background-color: #f8fafc;
            user-select: none;
            overflow-x: hidden;
        }
        canvas {
            touch-action: none;
        }
        .btn-glow {
            box-shadow: 0 0 15px rgba(217, 119, 6, 0.5);
        }
        .glass-panel {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(8px);
        }
    </style>
</head>
<body class="text-slate-800 min-h-screen flex flex-col">

    <header class="bg-amber-950 text-amber-50 shadow-lg py-3 px-4 md:px-6 border-b-4 border-amber-600 sticky top-0 z-50">
        <div class="max-w-7xl mx-auto flex flex-col md:flex-row justify-between items-center gap-2">
            <div class="flex items-center gap-3">
                <i class="fa-solid fa-tree text-2xl md:text-3xl text-emerald-400"></i>
                <div>
                    <h1 class="text-lg md:text-xl font-extrabold tracking-wide">木材切り出しシミュレーター 3D</h1>
                    <p class="text-[11px] text-amber-200">立木 → 丸太 → 板（柾目・板目・木表・木裏・反り）学習ツール</p>
                </div>
            </div>

            <!-- 3ステップ インジケーター -->
            <div class="flex items-center gap-1.5 md:gap-2 bg-amber-900/80 px-3 py-1.5 rounded-xl border border-amber-700/60 text-xs font-bold shadow-inner">
                <span id="stepBadge1" class="px-2.5 py-1 rounded-lg bg-amber-500 text-slate-950 transition-all">① 立木から丸太（2点指定）</span>
                <i class="fa-solid fa-chevron-right text-amber-400 text-[10px]"></i>
                <span id="stepBadge2" class="px-2.5 py-1 rounded-lg bg-amber-950/60 text-amber-300/60 transition-all">② 板切り出し（2点指定）</span>
                <i class="fa-solid fa-chevron-right text-amber-400 text-[10px]"></i>
                <span id="stepBadge3" class="px-2.5 py-1 rounded-lg bg-amber-950/60 text-amber-300/60 transition-all">③ 3D板の木目・反り観察</span>
            </div>
        </div>
    </header>

    <main class="flex-grow max-w-6xl w-full mx-auto p-3 md:p-6 flex flex-col justify-center">

        <!-- ステップ①：1本の立木から丸太（玉切り）の切り出し -->
        <div id="screenStep1" class="space-y-4">
            <div class="bg-white p-4 md:p-6 rounded-2xl shadow-sm border border-slate-200 grid grid-cols-1 lg:grid-cols-12 gap-6">
                
                <!-- 3D 樹木表示ビューアー -->
                <div class="lg:col-span-7 flex flex-col space-y-3">
                    <div class="flex justify-between items-center">
                        <h2 class="text-base md:text-lg font-bold text-slate-800 flex items-center gap-2">
                            <span class="bg-emerald-700 text-white w-7 h-7 rounded-full flex items-center justify-center text-xs font-bold">1</span>
                            立木の切断位置（点A・点B）を指定
                        </h2>
                        <span class="text-xs text-slate-500"><i class="fa-solid fa-hand-pointer text-emerald-600"></i> ドラッグで木を回転</span>
                    </div>

                    <div class="relative w-full h-[380px] bg-gradient-to-b from-sky-200 via-sky-100 to-emerald-100 rounded-2xl overflow-hidden shadow-inner border border-slate-300">
                        <div id="treeContainer" class="w-full h-full cursor-grab active:cursor-grabbing"></div>
                        
                        <!-- 切り出し位置ガイドラベル -->
                        <div class="absolute top-3 left-3 bg-slate-900/80 text-white text-xs px-3 py-1.5 rounded-lg backdrop-blur-sm space-y-1">
                            <div class="flex items-center gap-2">
                                <span class="w-3 h-3 rounded-full bg-red-500 inline-block"></span>
                                <span>点 A (上切断): <strong id="treeCutALabelText" class="text-red-300">4.5 m</strong></span>
                            </div>
                            <div class="flex items-center gap-2">
                                <span class="w-3 h-3 rounded-full bg-blue-500 inline-block"></span>
                                <span>点 B (下切断): <strong id="treeCutBLabelText" class="text-blue-300">1.5 m</strong></span>
                            </div>
                        </div>

                        <div class="absolute bottom-3 right-3 bg-slate-900/70 text-slate-200 text-[11px] px-3 py-1 rounded-lg backdrop-blur-sm">
                            樹高: 約15m（スギ・ヒノキ等）
                        </div>
                    </div>
                </div>

                <!-- 右側操作パネル -->
                <div class="lg:col-span-5 flex flex-col justify-between space-y-4">
                    <div class="bg-slate-50 p-4 rounded-xl border border-slate-200 space-y-4">
                        <h3 class="text-sm font-bold text-slate-800 border-b pb-2 flex items-center gap-2">
                            <i class="fa-solid fa-scissors text-emerald-700"></i> チェンソーで切断する2位置の設定
                        </h3>

                        <!-- 点A：上側切断位置スライダー -->
                        <div class="space-y-1">
                            <div class="flex justify-between items-center text-xs font-bold text-slate-700">
                                <span class="flex items-center gap-1.5 text-red-600">
                                    <span class="w-2.5 h-2.5 rounded-full bg-red-500 inline-block"></span> 点 A（上の切断線）
                                </span>
                                <span id="treeCutALabel" class="text-red-700 bg-red-50 px-2 py-0.5 rounded font-extrabold text-xs border border-red-200">4.5 m</span>
                            </div>
                            <input type="range" id="treeCutASlider" min="2.0" max="8.0" step="0.2" value="4.5" class="w-full accent-red-600 h-2 bg-slate-200 rounded-lg cursor-pointer">
                        </div>

                        <!-- 点B：下側切断位置スライダー -->
                        <div class="space-y-1">
                            <div class="flex justify-between items-center text-xs font-bold text-slate-700">
                                <span class="flex items-center gap-1.5 text-blue-600">
                                    <span class="w-2.5 h-2.5 rounded-full bg-blue-500 inline-block"></span> 点 B（下の切断線）
                                </span>
                                <span id="treeCutBLabel" class="text-blue-700 bg-blue-50 px-2 py-0.5 rounded font-extrabold text-xs border border-blue-200">1.5 m</span>
                            </div>
                            <input type="range" id="treeCutBSlider" min="0.5" max="6.5" step="0.2" value="1.5" class="w-full accent-blue-600 h-2 bg-slate-200 rounded-lg cursor-pointer">
                        </div>

                        <!-- 切り出す丸太の合計長さ -->
                        <div class="bg-amber-100 p-3 rounded-lg border border-amber-300 flex justify-between items-center">
                            <span class="text-xs font-bold text-amber-900">切り出される丸太の長さ（A - B）:</span>
                            <span id="logLengthLabel" class="text-amber-950 font-extrabold text-base bg-amber-200 px-2.5 py-0.5 rounded shadow-sm">3.0 m</span>
                        </div>

                        <div class="bg-emerald-50 p-3 rounded-lg border border-emerald-200 text-xs text-emerald-900 space-y-1">
                            <span class="font-bold block"><i class="fa-solid fa-lightbulb text-emerald-600"></i> ポイント:</span>
                            <p>幹の根元近く（点Bが低い位置）は幹が太く、まっすぐで質の高い建築用部材を切り出すことができます。</p>
                        </div>
                    </div>

                    <button id="btnCutTree" onclick="cutTreeToLog()" class="w-full bg-gradient-to-r from-emerald-600 to-teal-800 hover:from-emerald-700 hover:to-teal-900 text-white font-bold py-4 px-6 rounded-xl shadow-lg btn-glow transition flex items-center justify-center gap-3 text-base">
                        <i class="fa-solid fa-scissors text-emerald-300 text-xl"></i>
                        <span>この2点で丸太を切り出す！（チェンソー切断）</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- ステップ②：丸太の断面から2点切断（板の切り出し） -->
        <div id="screenStep2" class="space-y-4 hidden">
            
            <div class="flex justify-between items-center bg-white p-3 rounded-xl border border-slate-200 shadow-sm">
                <button onclick="goToStep(1)" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-2 rounded-lg transition flex items-center gap-1.5">
                    <i class="fa-solid fa-arrow-left"></i> 立木選びに戻る（画面①へ）
                </button>
                <span class="text-xs text-slate-500 font-bold">
                    <i class="fa-solid fa-circle-info text-amber-600"></i> 丸太から板（製材）の位置を2点で決めます
                </span>
            </div>

            <!-- コントロール & プリセットバー -->
            <div class="bg-white p-4 rounded-xl shadow-sm border border-slate-200 flex flex-wrap items-center justify-between gap-3">
                <div class="flex flex-wrap items-center gap-2">
                    <span class="text-xs font-bold text-slate-700 mr-1">
                        <i class="fa-solid fa-wand-magic-sparkles text-amber-600"></i> 切り出しプリセット:
                    </span>
                    <button onclick="setPreset('itame')" class="bg-orange-100 hover:bg-orange-200 text-orange-900 border border-orange-300 px-3 py-1 rounded-lg text-xs font-bold transition flex items-center gap-1">
                        <i class="fa-solid fa-water text-orange-700"></i> 板目（いため・外側）
                    </button>
                    <button onclick="setPreset('masame')" class="bg-amber-100 hover:bg-amber-200 text-amber-900 border border-amber-300 px-3 py-1 rounded-lg text-xs font-bold transition flex items-center gap-1">
                        <i class="fa-solid fa-grip-lines-vertical text-amber-700"></i> 柾目（まさめ・中心寄り）
                    </button>
                    <button onclick="setPreset('center')" class="bg-slate-100 hover:bg-slate-200 text-slate-800 border border-slate-300 px-3 py-1 rounded-lg text-xs transition">
                        樹心（中心通り）
                    </button>
                </div>

                <div class="flex items-center gap-3 text-xs font-bold text-slate-600">
                    <div class="flex items-center gap-1">
                        <span class="w-3 h-3 rounded-full bg-red-500 inline-block"></span>
                        <span>点 A</span>
                    </div>
                    <div class="flex items-center gap-1">
                        <span class="w-3 h-3 rounded-full bg-blue-500 inline-block"></span>
                        <span>点 B</span>
                    </div>
                </div>
            </div>

            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 grid grid-cols-1 lg:grid-cols-12 gap-6 items-center">
                
                <!-- 丸太断面 Canvas 指定エリア -->
                <div class="lg:col-span-6 flex flex-col items-center">
                    <div class="w-full text-center mb-2">
                        <h2 class="text-base font-bold text-slate-800 flex items-center justify-center gap-2">
                            <span class="bg-amber-800 text-white w-6 h-6 rounded-full flex items-center justify-center text-xs font-bold">2</span>
                            丸太断面の切断ライン（2点）を指定
                        </h2>
                        <p class="text-[11px] text-slate-500">
                            断面の<strong class="text-red-600">点A</strong>と<strong class="text-blue-600">点B</strong>をドラッグして切断位置を動かしてください。
                        </p>
                    </div>

                    <div class="relative w-full max-w-[360px] aspect-square bg-amber-50/50 rounded-2xl border-2 border-amber-200 overflow-hidden flex items-center justify-center shadow-inner">
                        <canvas id="logCanvas" class="w-full h-full cursor-pointer"></canvas>
                    </div>
                </div>

                <!-- 厚み設定 ＆ アニメーション実行 -->
                <div class="lg:col-span-6 space-y-5">
                    <div class="bg-amber-50/90 p-4 rounded-xl border border-amber-300 shadow-sm space-y-3">
                        <div class="flex justify-between items-center text-xs font-bold text-slate-700">
                            <span class="flex items-center gap-1.5 text-amber-900 text-sm">
                                <i class="fa-solid fa-ruler-combined text-amber-700"></i> 切り出す板の高さ（厚み）
                            </span>
                            <span id="thickVal" class="text-amber-800 font-extrabold text-base bg-amber-200/60 px-2 py-0.5 rounded">20 mm</span>
                        </div>
                        <input type="range" id="thickSlider" min="10" max="50" value="20" class="w-full accent-amber-700 h-2 bg-amber-200/50 rounded-lg cursor-pointer">
                        <div class="flex justify-between text-[10px] text-slate-500 font-medium">
                            <span>薄い（10mm）</span>
                            <span>標準（25mm）</span>
                            <span>厚い / 高い（50mm）</span>
                        </div>
                    </div>

                    <div id="grainPredictBox" class="p-3.5 rounded-xl border text-xs leading-relaxed space-y-1">
                        <!-- 動的予測計算テキスト -->
                    </div>

                    <button id="cutBtn" onclick="startCutAnimation()" class="w-full bg-gradient-to-r from-amber-600 to-amber-800 hover:from-amber-700 hover:to-amber-900 text-white font-bold py-3.5 px-6 rounded-xl shadow-lg btn-glow transition flex items-center justify-center gap-3 text-base">
                        <i class="fa-solid fa-scissors text-amber-300 text-xl"></i>
                        <span>この2点で板を切り出す！（製材アニメーション）</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- ステップ③：切り出した板の3D表面観察＆反りアニメーション -->
        <div id="screenStep3" class="space-y-4 hidden">
            
            <div class="flex justify-between items-center bg-white p-3 rounded-xl border border-slate-200 shadow-sm">
                <button onclick="goToStep(2)" class="bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold px-3 py-1.5 rounded-lg text-xs transition flex items-center gap-1.5">
                    <i class="fa-solid fa-arrow-left"></i> 別の位置で切り直す（画面②へ）
                </button>

                <div class="flex items-center gap-2">
                    <span id="grainBadge" class="text-xs font-bold px-3 py-1 rounded-full bg-orange-100 text-orange-800 border border-orange-300">
                        板目（いため）
                    </span>
                    <button id="warpToggleBtn" onclick="toggleWarpAnimation()" class="bg-cyan-600 hover:bg-cyan-700 text-white text-xs px-3.5 py-1.5 rounded-lg font-bold transition flex items-center gap-1.5 shadow-sm">
                        <i class="fa-solid fa-wind"></i> 乾燥テスト（反り発生）
                    </button>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-12 gap-5">
                
                <!-- 3D ビューアー メインエリア -->
                <div class="lg:col-span-8 bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center mb-3">
                            <h2 class="text-base font-bold text-slate-800 flex items-center gap-2">
                                <span class="bg-amber-800 text-white w-6 h-6 rounded-full flex items-center justify-center text-xs">3</span>
                                切り出した板の3D観察（ドラッグで360度回転）
                            </h2>
                            <span class="text-xs text-slate-500"><i class="fa-solid fa-hand-pointer text-amber-600"></i> マウス・指で回転/拡大</span>
                        </div>

                        <!-- 視点切り替えボタン群 -->
                        <div class="flex flex-wrap gap-2 mb-3">
                            <button onclick="setCameraView('perspective')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold px-3 py-1 rounded-lg border border-slate-300">
                                <i class="fa-solid fa-cube mr-1"></i> 全体斜め
                            </button>
                            <button onclick="setCameraView('koguchi')" class="bg-amber-100 hover:bg-amber-200 text-amber-900 text-xs font-bold px-3 py-1 rounded-lg border border-amber-300">
                                <i class="fa-solid fa-circle-dot mr-1"></i> 木口（こぐち・年輪）
                            </button>
                            <button onclick="setCameraView('kio')" class="bg-orange-100 hover:bg-orange-200 text-orange-900 text-xs font-bold px-3 py-1 rounded-lg border border-orange-300">
                                <i class="fa-solid fa-sun mr-1"></i> 木表（きおもて）
                            </button>
                            <button onclick="setCameraView('kiu')" class="bg-slate-200 hover:bg-slate-300 text-slate-800 text-xs font-bold px-3 py-1 rounded-lg border border-slate-400">
                                <i class="fa-solid fa-moon mr-1"></i> 木裏（きうら）
                            </button>
                        </div>

                        <!-- 3D Canvas コンテナ -->
                        <div class="relative w-full h-[360px] bg-slate-900 rounded-2xl overflow-hidden shadow-inner border-2 border-slate-700">
                            <div id="boardContainer" class="w-full h-full cursor-grab active:cursor-grabbing"></div>
                            
                            <div id="warpStatusTag" class="absolute top-3 left-3 bg-slate-800/90 text-cyan-300 text-xs px-3 py-1 rounded-lg border border-cyan-500/40 font-bold backdrop-blur-sm">
                                状態: 未乾燥（直線の板）
                            </div>

                            <div class="absolute bottom-3 right-3 bg-slate-800/80 text-slate-300 text-[11px] px-3 py-1 rounded-lg border border-slate-600">
                                ドラッグで回転 / スクロールで拡大
                            </div>
                        </div>
                    </div>

                    <!-- 3D用語ガイド -->
                    <div class="mt-4 grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-amber-50 border border-amber-200 p-2 rounded-xl">
                            <span class="font-bold text-amber-900 block">【木口（こぐち）】</span>
                            <span class="text-slate-600 text-[10px]">両端の年輪が見える切断面</span>
                        </div>
                        <div class="bg-orange-50 border border-orange-200 p-2 rounded-xl">
                            <span class="font-bold text-orange-900 block">【木表（きおもて）】</span>
                            <span class="text-slate-600 text-[10px]">樹皮に近い外側の面（山形模様）</span>
                        </div>
                        <div class="bg-slate-100 border border-slate-300 p-2 rounded-xl">
                            <span class="font-bold text-slate-800 block">【木裏（きうら）】</span>
                            <span class="text-slate-600 text-[10px]">樹心に近い内側の面</span>
                        </div>
                    </div>
                </div>

                <!-- 右側：解説カード ＆ サマリー -->
                <div class="lg:col-span-4 space-y-4 flex flex-col justify-between">
                    
                    <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-200 space-y-3">
                        <h3 class="font-bold text-slate-800 border-b pb-2 flex items-center gap-2 text-sm">
                            <i class="fa-solid fa-circle-info text-amber-700"></i> 木表・木裏の乾燥・反り法則
                        </h3>
                        
                        <div class="space-y-2 text-xs text-slate-700 leading-relaxed">
                            <div class="bg-amber-50/80 p-3 rounded-xl border border-amber-200">
                                <span class="font-bold text-amber-900 block mb-1">☀️ 木表（樹皮側・外側）</span>
                                水分が多く収縮率が大きいため、乾燥すると<strong>木表側に凹（へこむ）ように反ります。</strong>
                            </div>

                            <div class="bg-slate-50 p-3 rounded-xl border border-slate-200">
                                <span class="font-bold text-slate-800 block mb-1">🌙 木裏（樹心側・内側）</span>
                                水分が少なく収縮が穏やかなため、乾燥時は凸（膨らむ）側になります。
                            </div>
                        </div>
                    </div>

                    <!-- 特徴サマリー -->
                    <div id="featureSummary" class="bg-amber-900 text-amber-50 p-4 rounded-2xl shadow-sm space-y-2">
                        <!-- JSで動的更新 -->
                    </div>

                </div>
            </div>
        </div>
    </main>

    <footer class="bg-slate-800 text-slate-400 text-xs py-3 text-center mt-auto border-t border-slate-700">
        <p>中学校技術科 木材加工学習ツール 〜 立木・丸太・板の3D切り出し＆木表・木裏観察 〜</p>
    </footer>

    <script>
        // ==========================================
        // グローバル状態管理
        // ==========================================
        const state = {
            currentStep: 1,       // 1: 立木切断, 2: 丸太切断(2点), 3: 3D板観察
            
            // ステップ1: 立木から丸太（2点切断）
            treeCutA: 4.5,        // 点A 上切断位置 (m)
            treeCutB: 1.5,        // 点B 下切断位置 (m)
            logLength: 3.0,       // メートル (A - B)
            
            // ステップ2: 丸太から板（丸太断面座標系）
            pointA: { x: -110, y: -70 },
            pointB: { x: 110, y: -70 },
            thickness: 20,        // 板の高さ/厚み (mm)
            logRadius: 135,       // 画面上の丸太半径 (px)
            rings: 11,            // 年輪数

            // アニメーションフラグ
            isTreeCutting: false,
            isBoardCutting: false,
            cutProgress: 0,
            particles: [],

            // 反り状態
            isWarped: false,
            warpProgress: 0,
            warpAnimId: null,

            // ドラッグ操作
            activePoint: null,
            isDragging: false
        };

        // Canvas & Ctx
        let logCanvas, logCtx;

        // Three.js 空間インスタンス
        let treeScene, treeCamera, treeRenderer, treeControls, treeTrunkMesh;
        let cutRingA, cutRingB, cutHighlightMesh;
        let boardScene, boardCamera, boardRenderer, boardControls, boardMesh;
        let originalVertices = [];

        let chainsawMesh = null;
        let treeParticlesArr = [];

        // 初期化処理
        window.onload = function() {
            logCanvas = document.getElementById('logCanvas');
            if (logCanvas) logCtx = logCanvas.getContext('2d');

            // 画面リサイズリスナー
            window.addEventListener('resize', handleResize);

            // コントロールリスナー
            setupControls();
            setupInteractivePoints();

            // 3D 空間初期化
            initTree3D();
            initBoard3D();

            // 初期位置設定 (板目設定)
            setPreset('itame');

            handleResize();
        };

        function handleResize() {
            const dpr = window.devicePixelRatio || 1;

            if (logCanvas) {
                const rect = logCanvas.getBoundingClientRect();
                logCanvas.width = rect.width * dpr;
                logCanvas.height = rect.height * dpr;
                if (logCtx) logCtx.scale(dpr, dpr);
            }

            if (treeRenderer && treeCamera) {
                const container = document.getElementById('treeContainer');
                if (container) {
                    treeCamera.aspect = container.clientWidth / container.clientHeight;
                    treeCamera.updateProjectionMatrix();
                    treeRenderer.setSize(container.clientWidth, container.clientHeight);
                }
            }

            if (boardRenderer && boardCamera) {
                const container = document.getElementById('boardContainer');
                if (container) {
                    boardCamera.aspect = container.clientWidth / container.clientHeight;
                    boardCamera.updateProjectionMatrix();
                    boardRenderer.setSize(container.clientWidth, container.clientHeight);
                }
            }

            updateAll();
        }

        // ==========================================
        // [3D Scene 1] 立木（Tree）のThree.jsセットアップ
        // ==========================================
        function initTree3D() {
            const container = document.getElementById('treeContainer');
            if (!container) return;

            treeScene = new THREE.Scene();
            treeScene.background = new THREE.Color(0xe0f2fe); // 青空背景

            treeCamera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
            treeCamera.position.set(0, 4, 18);

            treeRenderer = new THREE.WebGLRenderer({ antialias: true });
            treeRenderer.setSize(container.clientWidth, container.clientHeight);
            treeRenderer.setPixelRatio(window.devicePixelRatio);
            treeRenderer.shadowMap.enabled = true;
            container.appendChild(treeRenderer.domElement);

            treeControls = new THREE.OrbitControls(treeCamera, treeRenderer.domElement);
            treeControls.enableDamping = true;
            treeControls.dampingFactor = 0.05;
            treeControls.maxPolarAngle = Math.PI / 2 + 0.1; // 地面より下に行かない

            // ライト
            const ambLight = new THREE.AmbientLight(0xffffff, 0.7);
            treeScene.add(ambLight);

            const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
            dirLight.position.set(10, 20, 10);
            dirLight.castShadow = true;
            treeScene.add(dirLight);

            // 地面（草地）
            const groundGeo = new THREE.PlaneGeometry(50, 50);
            const groundMat = new THREE.MeshStandardMaterial({ color: 0x86efac, roughness: 0.9 });
            const ground = new THREE.Mesh(groundGeo, groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.position.y = -5;
            ground.receiveShadow = true;
            treeScene.add(ground);

            // 樹木の構築 (幹 + 葉)
            buildTreeModel();

            // レンダリングループ
            const animateTree = () => {
                requestAnimationFrame(animateTree);
                if (treeControls) treeControls.update();
                updateTreeParticles();
                if (treeRenderer && treeScene && treeCamera) {
                    treeRenderer.render(treeScene, treeCamera);
                }
            };
            animateTree();
        }

        function createChainsawMesh() {
            const group = new THREE.Group();

            // チェンソー本体 (橙色のエンジンボディ)
            const bodyGeo = new THREE.BoxGeometry(0.7, 0.45, 0.45);
            const bodyMat = new THREE.MeshStandardMaterial({ color: 0xea580c, roughness: 0.3 });
            const body = new THREE.Mesh(bodyGeo, bodyMat);
            body.position.set(-0.3, 0, 0);
            group.add(body);

            // ハンドル
            const handleGeo = new THREE.TorusGeometry(0.22, 0.04, 8, 16, Math.PI * 1.2);
            const handleMat = new THREE.MeshStandardMaterial({ color: 0x1e293b });
            const handle = new THREE.Mesh(handleGeo, handleMat);
            handle.rotation.y = Math.PI / 2;
            handle.position.set(-0.3, 0.22, 0);
            group.add(handle);

            // ガイドバー (シルバーの板)
            const barGeo = new THREE.BoxGeometry(1.2, 0.2, 0.04);
            const barMat = new THREE.MeshStandardMaterial({ color: 0xcbd5e1, metalness: 0.8, roughness: 0.2 });
            const bar = new THREE.Mesh(barGeo, barMat);
            bar.position.set(0.6, 0, 0);
            group.add(bar);

            // ソーチェーン (刃)
            const chainGeo = new THREE.BoxGeometry(1.24, 0.23, 0.02);
            const chainMat = new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.9, roughness: 0.1 });
            const chain = new THREE.Mesh(chainGeo, chainMat);
            chain.position.set(0.62, 0, 0);
            group.add(chain);

            group.visible = false;
            return group;
        }

        function spawnTreeParticles(x, y, z, count = 4) {
            if (!treeScene) return;
            for (let i = 0; i < count; i++) {
                const geo = new THREE.BoxGeometry(0.07, 0.07, 0.07);
                const mat = new THREE.MeshBasicMaterial({
                    color: Math.random() > 0.35 ? 0xf59e0b : 0x78350f
                });
                const p = new THREE.Mesh(geo, mat);
                p.position.set(x + (Math.random() - 0.5) * 0.2, y + (Math.random() - 0.5) * 0.1, z + (Math.random() - 0.5) * 0.3);
                
                p.userData = {
                    vx: (Math.random() - 0.5) * 0.12,
                    vy: Math.random() * 0.08 + 0.03,
                    vz: (Math.random() - 0.5) * 0.12 + 0.06,
                    life: 1.0
                };
                treeScene.add(p);
                treeParticlesArr.push(p);
            }
        }

        function updateTreeParticles() {
            for (let i = treeParticlesArr.length - 1; i >= 0; i--) {
                const p = treeParticlesArr[i];
                p.position.x += p.userData.vx;
                p.position.y += p.userData.vy;
                p.position.z += p.userData.vz;
                p.userData.vy -= 0.006; // 重力で落下
                p.userData.life -= 0.035;
                p.scale.multiplyScalar(0.94);

                if (p.userData.life <= 0) {
                    treeScene.remove(p);
                    p.geometry.dispose();
                    p.material.dispose();
                    treeParticlesArr.splice(i, 1);
                }
            }
        }

        function buildTreeModel() {
            const treeGroup = new THREE.Group();

            // 幹 (Trunk) - 高さ14m
            const trunkGeo = new THREE.CylinderGeometry(0.9, 1.3, 14, 24);
            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x78350f, roughness: 0.8 });
            treeTrunkMesh = new THREE.Mesh(trunkGeo, trunkMat);
            treeTrunkMesh.position.y = 2; // 中心y=2 (底y=-5 〜 頂点y=9)
            treeTrunkMesh.castShadow = true;
            treeGroup.add(treeTrunkMesh);

            // 葉 (Leaves Cone)
            const leafMat = new THREE.MeshStandardMaterial({ color: 0x15803d, roughness: 0.6 });
            for (let i = 0; i < 3; i++) {
                const leafGeo = new THREE.ConeGeometry(3.5 - i * 0.6, 5, 16);
                const leaf = new THREE.Mesh(leafGeo, leafMat);
                leaf.position.y = 5.5 + i * 2.2;
                leaf.castShadow = true;
                treeGroup.add(leaf);
            }

            // 2点の切断線ガイドリング（点A: 赤, 点B: 青）
            const ringGeo = new THREE.TorusGeometry(1.4, 0.08, 16, 40);
            
            const ringMatA = new THREE.MeshBasicMaterial({ color: 0xef4444 });
            cutRingA = new THREE.Mesh(ringGeo, ringMatA);
            cutRingA.rotation.x = Math.PI / 2;
            treeGroup.add(cutRingA);

            const ringMatB = new THREE.MeshBasicMaterial({ color: 0x3b82f6 });
            cutRingB = new THREE.Mesh(ringGeo, ringMatB);
            cutRingB.rotation.x = Math.PI / 2;
            treeGroup.add(cutRingB);

            // 切断選択範囲のハイライト円筒
            const hlGeo = new THREE.CylinderGeometry(1.2, 1.2, 1, 24);
            const hlMat = new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.DoubleSide });
            cutHighlightMesh = new THREE.Mesh(hlGeo, hlMat);
            treeGroup.add(cutHighlightMesh);

            treeScene.add(treeGroup);
            updateTreeCutVisuals();
        }

        // 立木の2点切断リングおよびハイライト表示の更新
        function updateTreeCutVisuals() {
            if (!cutRingA || !cutRingB || !cutHighlightMesh) return;

            const yA = -5 + state.treeCutA;
            const yB = -5 + state.treeCutB;

            cutRingA.position.y = yA;
            cutRingB.position.y = yB;

            const midY = (yA + yB) / 2;
            const height = Math.abs(yA - yB);

            cutHighlightMesh.position.y = midY;
            cutHighlightMesh.scale.set(1, Math.max(0.01, height), 1);
        }

        // ==========================================
        // [3D Scene 2] 板（Board）のThree.jsセットアップ
        // ==========================================
        function initBoard3D() {
            const container = document.getElementById('boardContainer');
            if (!container) return;

            boardScene = new THREE.Scene();
            boardScene.background = new THREE.Color(0x0f172a); // ダーク背景

            boardCamera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
            boardCamera.position.set(0, 8, 18);

            boardRenderer = new THREE.WebGLRenderer({ antialias: true });
            boardRenderer.setSize(container.clientWidth, container.clientHeight);
            boardRenderer.setPixelRatio(window.devicePixelRatio);
            boardRenderer.shadowMap.enabled = true;
            container.appendChild(boardRenderer.domElement);

            boardControls = new THREE.OrbitControls(boardCamera, boardRenderer.domElement);
            boardControls.enableDamping = true;
            boardControls.dampingFactor = 0.05;

            // ライティング
            const ambLight = new THREE.AmbientLight(0xffffff, 0.7);
            boardScene.add(ambLight);

            const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
            dirLight.position.set(10, 20, 15);
            dirLight.castShadow = true;
            boardScene.add(dirLight);

            const fillLight = new THREE.DirectionalLight(0xfef3c7, 0.4);
            fillLight.position.set(-10, -10, -10);
            boardScene.add(fillLight);

            // レンダリングループ
            const animateBoard = () => {
                requestAnimationFrame(animateBoard);
                if (boardControls) boardControls.update();
                if (boardRenderer && boardScene && boardCamera) {
                    boardRenderer.render(boardScene, boardCamera);
                }
            };
            animateBoard();
        }

        // 視点切り替え
        function setCameraView(view) {
            if (!boardCamera || !boardControls) return;

            let targetPos = { x: 0, y: 8, z: 18 };
            if (view === 'koguchi') targetPos = { x: 18, y: 0, z: 0 };
            else if (view === 'kio') targetPos = { x: 0, y: 18, z: 0.1 };
            else if (view === 'kiu') targetPos = { x: 0, y: -18, z: 0.1 };

            const startPos = { x: boardCamera.position.x, y: boardCamera.position.y, z: boardCamera.position.z };
            const startTime = Date.now();
            const duration = 550;

            const updateCam = () => {
                const elapsed = Date.now() - startTime;
                const progress = Math.min(1.0, elapsed / duration);
                const ease = 1 - Math.pow(1 - progress, 3);

                boardCamera.position.x = startPos.x + (targetPos.x - startPos.x) * ease;
                boardCamera.position.y = startPos.y + (targetPos.y - startPos.y) * ease;
                boardCamera.position.z = startPos.z + (targetPos.z - startPos.z) * ease;
                boardControls.target.set(0, 0, 0);

                if (progress < 1.0) requestAnimationFrame(updateCam);
            };
            updateCam();
        }

        // 3D板モデルの構築
        function build3DBoard(info) {
            if (!boardScene) return;

            if (boardMesh) {
                boardScene.remove(boardMesh);
                boardMesh.geometry.dispose();
                if (Array.isArray(boardMesh.material)) {
                    boardMesh.material.forEach(m => m.dispose());
                }
            }

            const length = 12.0;
            const width = (info.len / state.logRadius) * 6.5;
            const thick = (state.thickness / 40.0) * 1.2;

            // テクスチャ生成
            const kioCanvas = generateSurfaceCanvas(info, 'kio');
            const kiuCanvas = generateSurfaceCanvas(info, 'kiu');
            const koguchiCanvas = generateKoguchiCanvas(info);
            const sideCanvas = generateSideCanvas();

            const materials = [
                new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(koguchiCanvas), roughness: 0.8 }),
                new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(koguchiCanvas), roughness: 0.8 }),
                new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(kioCanvas), roughness: 0.6 }),
                new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(kiuCanvas), roughness: 0.6 }),
                new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(sideCanvas), roughness: 0.7 }),
                new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(sideCanvas), roughness: 0.7 })
            ];

            const geometry = new THREE.BoxGeometry(width, thick, length, 30, 5, 1);
            boardMesh = new THREE.Mesh(geometry, materials);
            boardMesh.castShadow = true;
            boardMesh.receiveShadow = true;

            originalVertices = [];
            const posAttr = geometry.attributes.position;
            for (let i = 0; i < posAttr.count; i++) {
                originalVertices.push({ x: posAttr.getX(i), y: posAttr.getY(i), z: posAttr.getZ(i) });
            }

            boardScene.add(boardMesh);
            apply3DWarp(info);
        }

        // 動的テクスチャ生成 (Canvas 2D)
        function generateSurfaceCanvas(info, side) {
            const canvas = document.createElement('canvas');
            canvas.width = 512; canvas.height = 512;
            const ctx = canvas.getContext('2d');
            const w = canvas.width, h = canvas.height;

            // 木肌のベースカラー（自然なオーク・スギ材の質感）
            const bgGrad = ctx.createLinearGradient(0, 0, w, 0);
            bgGrad.addColorStop(0, "#f7ebdb");
            bgGrad.addColorStop(0.5, "#e3c9a8");
            bgGrad.addColorStop(1, "#f7ebdb");
            ctx.fillStyle = bgGrad; ctx.fillRect(0, 0, w, h);

            const isKio = side === 'kio';
            const cx = w * 0.5;

            if (info.isMasame) {
                // 【柾目】 樹心近くを通る平行な直線状の木目
                ctx.strokeStyle = "rgba(133, 62, 17, 0.65)";
                ctx.lineWidth = 3.0;
                const spacing = 18 + (info.absDist * 0.15);
                for (let x = -20; x <= w + 20; x += spacing) {
                    ctx.beginPath();
                    for (let y = 0; y <= h; y += 16) {
                        const wave = Math.sin(y * 0.015 + x * 0.05) * 3;
                        if (y === 0) ctx.moveTo(x + wave, y);
                        else ctx.lineTo(x + wave, y);
                    }
                    ctx.stroke();
                }
            } else {
                // 【板目】 全域に隙間なく埋め尽くす同心アーチ（筍目＆外側縦木目）
                ctx.strokeStyle = "rgba(110, 45, 10, 0.85)";
                ctx.lineWidth = 2.6;

                const dir = (info.signedDist >= 0) ? 1 : -1;
                const archDir = isKio ? dir : -dir; // 1: 山が上向き (▲), -1: 山が下向き (▼)

                // 切断線の丸太中心からの距離 d
                const d = Math.max(12, info.absDist * 1.05);

                // 樹木のテーパー率（高さ方向に対する年輪半径の変化）
                const taper = 0.35 * archDir;
                const yRef = (archDir > 0) ? h * 0.72 : h * 0.28;

                // 年輪のピッチ（間隔）
                const pitch = 20;

                // キャンバス全域を完全にカバーする年輪半径 R_base のループ
                const minR = d - 80;
                const maxR = Math.hypot(w, h) + 250;

                for (let R_base = minR; R_base <= maxR; R_base += pitch) {
                    const validPoints = [];

                    // 高さ y (0 〜 h) をスキャンして年輪の輪郭位置 dx を算出
                    for (let y = 0; y <= h; y += 3) {
                        const Ry = R_base - taper * (y - yRef);
                        if (Ry > d) {
                            const rawVal = Ry * Ry - d * d;
                            const wave = Math.sin(y * 0.02 + R_base * 0.08) * 1.5;
                            const dx = Math.sqrt(rawVal) * 0.82 + wave;
                            validPoints.push({ y, dx });
                        }
                    }

                    if (validPoints.length < 2) continue;

                    const startsAtTop = (validPoints[0].y === 0);
                    const endsAtBottom = (validPoints[validPoints.length - 1].y === h);

                    if (startsAtTop && endsAtBottom) {
                        // 画面の上端から下端まで突き抜ける大外の年輪：左右それぞれ独立した縦線状アーチ
                        ctx.beginPath();
                        ctx.moveTo(cx - validPoints[0].dx, validPoints[0].y);
                        for (let i = 1; i < validPoints.length; i++) {
                            ctx.lineTo(cx - validPoints[i].dx, validPoints[i].y);
                        }
                        ctx.stroke();

                        ctx.beginPath();
                        ctx.moveTo(cx + validPoints[0].dx, validPoints[0].y);
                        for (let i = 1; i < validPoints.length; i++) {
                            ctx.lineTo(cx + validPoints[i].dx, validPoints[i].y);
                        }
                        ctx.stroke();
                    } else {
                        // 途中で頂点を迎える中〜小の年輪：山形アーチ（筍目）
                        ctx.beginPath();
                        ctx.moveTo(cx - validPoints[0].dx, validPoints[0].y);
                        for (let i = 1; i < validPoints.length; i++) {
                            ctx.lineTo(cx - validPoints[i].dx, validPoints[i].y);
                        }
                        for (let i = validPoints.length - 1; i >= 0; i--) {
                            ctx.lineTo(cx + validPoints[i].dx, validPoints[i].y);
                        }
                        ctx.stroke();
                    }
                }
            }

            // ラベル表記
            ctx.fillStyle = "#612808";
            ctx.font = "bold 26px sans-serif";
            ctx.textAlign = "center";
            ctx.fillText(isKio ? "【 木表（きおもて）】" : "【 木裏（きうら）】", w / 2, isKio ? 45 : h - 25);

            return canvas;
        }

        function generateKoguchiCanvas(info) {
            const canvas = document.createElement('canvas');
            canvas.width = 512; canvas.height = 256;
            const ctx = canvas.getContext('2d');
            const w = canvas.width, h = canvas.height;

            ctx.fillStyle = "#f3dfc6"; ctx.fillRect(0, 0, w, h);
            const cx = w / 2;
            const dir = info.signedDist >= 0 ? 1 : -1;
            const cy = h / 2 + (dir * (info.absDist / state.logRadius) * h * 1.5);

            ctx.strokeStyle = "rgba(120, 53, 15, 0.6)"; ctx.lineWidth = 3;
            for (let r = 15; r < 800; r += 18) {
                ctx.beginPath(); ctx.arc(cx, cy, r, 0, Math.PI * 2); ctx.stroke();
            }

            ctx.fillStyle = "#78350f"; ctx.font = "bold 28px sans-serif";
            ctx.textAlign = "center"; ctx.fillText("【木口】", w / 2, 40);

            return canvas;
        }

        function generateSideCanvas() {
            const canvas = document.createElement('canvas');
            canvas.width = 256; canvas.height = 256;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = "#e2b388"; ctx.fillRect(0, 0, 256, 256);
            ctx.strokeStyle = "rgba(146, 64, 14, 0.3)"; ctx.lineWidth = 2;
            for (let y = 0; y < 256; y += 12) {
                ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(256, y); ctx.stroke();
            }
            return canvas;
        }

        // 反り変形 (木表側が凹)
        function apply3DWarp(info) {
            if (!boardMesh || !originalVertices.length) return;
            const posAttr = boardMesh.geometry.attributes.position;
            const progress = state.warpProgress;

            const maxCurvature = info.isMasame ? 0.003 : 0.06;
            const k = maxCurvature * progress;

            for (let i = 0; i < posAttr.count; i++) {
                const orig = originalVertices[i];
                const deltaY = k * Math.pow(orig.x, 2);
                posAttr.setXYZ(i, orig.x, orig.y + deltaY, orig.z);
            }

            posAttr.needsUpdate = true;
            boardMesh.geometry.computeVertexNormals();
        }

        // ==========================================
        // ステップ遷移制御
        // ==========================================
        function goToStep(step) {
            state.currentStep = step;
            const screen1 = document.getElementById('screenStep1');
            const screen2 = document.getElementById('screenStep2');
            const screen3 = document.getElementById('screenStep3');

            const b1 = document.getElementById('stepBadge1');
            const b2 = document.getElementById('stepBadge2');
            const b3 = document.getElementById('stepBadge3');

            screen1.classList.add('hidden');
            screen2.classList.add('hidden');
            screen3.classList.add('hidden');

            const activeClass = "px-2.5 py-1 rounded-lg bg-amber-500 text-slate-950 transition-all font-bold shadow";
            const inactiveClass = "px-2.5 py-1 rounded-lg bg-amber-950/60 text-amber-300/60 transition-all";

            b1.className = inactiveClass;
            b2.className = inactiveClass;
            b3.className = inactiveClass;

            if (step === 1) {
                screen1.classList.remove('hidden');
                b1.className = activeClass;
            } else if (step === 2) {
                screen2.classList.remove('hidden');
                b2.className = activeClass;
            } else if (step === 3) {
                screen3.classList.remove('hidden');
                b3.className = activeClass;

                state.isWarped = false;
                state.warpProgress = 0;
                document.getElementById('warpToggleBtn').innerHTML = `<i class="fa-solid fa-wind"></i> 乾燥テスト（反り発生）`;
                document.getElementById('warpStatusTag').innerText = "状態: 未乾燥（直線の板）";

                const info = getLineInfo();
                build3DBoard(info);
                setCameraView('perspective');
            }

            setTimeout(handleResize, 50);
        }

        // ==========================================
        // イベント & インタラクション設定
        // ==========================================
        function setupControls() {
            const treeCutASlider = document.getElementById('treeCutASlider');
            const treeCutBSlider = document.getElementById('treeCutBSlider');

            const updateTreeCutValues = () => {
                let valA = parseFloat(treeCutASlider.value);
                let valB = parseFloat(treeCutBSlider.value);

                // 点A（上）が点B（下）を下回らないように補正
                if (valA <= valB + 0.5) {
                    valA = valB + 0.5;
                    treeCutASlider.value = valA;
                }

                state.treeCutA = valA;
                state.treeCutB = valB;
                state.logLength = valA - valB;

                document.getElementById('treeCutALabel').innerText = `${state.treeCutA.toFixed(1)} m`;
                document.getElementById('treeCutBLabel').innerText = `${state.treeCutB.toFixed(1)} m`;
                document.getElementById('treeCutALabelText').innerText = `${state.treeCutA.toFixed(1)} m`;
                document.getElementById('treeCutBLabelText').innerText = `${state.treeCutB.toFixed(1)} m`;
                document.getElementById('logLengthLabel').innerText = `${state.logLength.toFixed(1)} m`;

                updateTreeCutVisuals();
            };

            if (treeCutASlider) treeCutASlider.addEventListener('input', updateTreeCutValues);
            if (treeCutBSlider) treeCutBSlider.addEventListener('input', updateTreeCutValues);

            const thickSlider = document.getElementById('thickSlider');
            if (thickSlider) {
                thickSlider.addEventListener('input', (e) => {
                    state.thickness = parseInt(e.target.value);
                    updateAll();
                });
            }
        }

        function setupInteractivePoints() {
            const getCanvasPos = (clientX, clientY) => {
                const rect = logCanvas.getBoundingClientRect();
                const cx = rect.width / 2;
                const cy = rect.height / 2;
                return { x: clientX - rect.left - cx, y: clientY - rect.top - cy };
            };

            const handleStart = (clientX, clientY) => {
                if (state.isBoardCutting || state.currentStep !== 2) return;
                const pos = getCanvasPos(clientX, clientY);
                const hitRadius = 28;

                const distA = Math.hypot(pos.x - state.pointA.x, pos.y - state.pointA.y);
                const distB = Math.hypot(pos.x - state.pointB.x, pos.y - state.pointB.y);

                if (distA < hitRadius) { state.activePoint = 'A'; state.isDragging = true; }
                else if (distB < hitRadius) { state.activePoint = 'B'; state.isDragging = true; }
            };

            const handleMove = (clientX, clientY) => {
                if (!state.isDragging || !state.activePoint) return;
                const pos = getCanvasPos(clientX, clientY);
                const maxDist = state.logRadius * 1.15;
                const d = Math.hypot(pos.x, pos.y);

                let targetX = pos.x, targetY = pos.y;
                if (d > maxDist) {
                    targetX = (pos.x / d) * maxDist;
                    targetY = (pos.y / d) * maxDist;
                }

                if (state.activePoint === 'A') { state.pointA.x = targetX; state.pointA.y = targetY; }
                else if (state.activePoint === 'B') { state.pointB.x = targetX; state.pointB.y = targetY; }

                updateAll();
            };

            const handleEnd = () => { state.isDragging = false; state.activePoint = null; };

            logCanvas.addEventListener('mousedown', (e) => handleStart(e.clientX, e.clientY));
            window.addEventListener('mousemove', (e) => handleMove(e.clientX, e.clientY));
            window.addEventListener('mouseup', handleEnd);

            logCanvas.addEventListener('touchstart', (e) => {
                if (e.touches.length > 0) handleStart(e.touches[0].clientX, e.touches[0].clientY);
            }, { passive: true });

            window.addEventListener('touchmove', (e) => {
                if (e.touches.length > 0) handleMove(e.touches[0].clientX, e.touches[0].clientY);
            }, { passive: true });

            window.addEventListener('touchend', handleEnd);
        }

        function setPreset(type) {
            if (type === 'masame') {
                state.pointA = { x: 25, y: -120 }; state.pointB = { x: 25, y: 120 };
            } else if (type === 'itame') {
                state.pointA = { x: -110, y: -70 }; state.pointB = { x: 110, y: -70 };
            } else if (type === 'center') {
                state.pointA = { x: -120, y: 0 }; state.pointB = { x: 120, y: 0 };
            }
            updateAll();
        }

        // ==========================================
        // アニメーション実行 (ステップ1 & 2)
        // ==========================================
        function cutTreeToLog() {
            if (state.isTreeCutting) return;
            state.isTreeCutting = true;

            const btn = document.getElementById('btnCutTree');
            btn.disabled = true;
            btn.innerHTML = `<i class="fa-solid fa-gear fa-spin text-amber-300 text-xl"></i> <span>チェンソーで切断中...</span>`;
            btn.className = "w-full bg-slate-500 text-white font-bold py-4 px-6 rounded-xl cursor-not-allowed flex items-center justify-center gap-3 text-base shadow-inner";

            // 3Dチェンソーの準備
            if (!chainsawMesh) {
                chainsawMesh = createChainsawMesh();
                treeScene.add(chainsawMesh);
            }
            chainsawMesh.visible = true;

            const yA = -5 + state.treeCutA;
            const yB = -5 + state.treeCutB;
            const origCamY = treeCamera.position.y;

            const duration = 2600; // アニメーション時間
            const startTime = Date.now();

            const animateCutSequence = () => {
                const elapsed = Date.now() - startTime;
                const progress = Math.min(1.0, elapsed / duration);

                if (progress < 0.45) {
                    // --- フェーズ1: 点B（下切断線）を左から右へチェンソーで横断切断 ---
                    const subProg = progress / 0.45;
                    const currentX = -3.6 + subProg * 7.2;
                    chainsawMesh.position.set(currentX, yB, 0.2);
                    chainsawMesh.rotation.set(0, 0, Math.sin(elapsed * 0.06) * 0.1);

                    // 幹の内部通過中に木くず発生＆カメラ微振動
                    if (currentX > -1.2 && currentX < 1.2) {
                        spawnTreeParticles(currentX + 0.3, yB, 0, 4);
                        treeCamera.position.y = origCamY + (Math.random() - 0.5) * 0.15;
                    } else {
                        treeCamera.position.y = origCamY;
                    }

                } else if (progress < 0.55) {
                    // --- フェーズ2: 点A（上切断線）の開始位置へチェンソーが高速移動 ---
                    const subProg = (progress - 0.45) / 0.1;
                    const currentY = yB + (yA - yB) * subProg;
                    chainsawMesh.position.set(3.6 - subProg * 7.2, currentY, 0.2);
                    treeCamera.position.y = origCamY;

                } else if (progress < 0.95) {
                    // --- フェーズ3: 点A（上切断線）を左から右へチェンソーで横断切断 ---
                    const subProg = (progress - 0.55) / 0.40;
                    const currentX = -3.6 + subProg * 7.2;
                    chainsawMesh.position.set(currentX, yA, 0.2);
                    chainsawMesh.rotation.set(0, 0, Math.sin(elapsed * 0.06) * 0.1);

                    if (currentX > -1.2 && currentX < 1.2) {
                        spawnTreeParticles(currentX + 0.3, yA, 0, 4);
                        treeCamera.position.y = origCamY + (Math.random() - 0.5) * 0.15;
                    } else {
                        treeCamera.position.y = origCamY;
                    }

                } else {
                    // --- フェーズ4: 切断終了・チェンソー格納 ---
                    chainsawMesh.visible = false;
                    treeCamera.position.y = origCamY;

                    if (cutHighlightMesh) {
                        cutHighlightMesh.material.color.setHex(0xfacc15); // フラッシュ
                    }
                }

                if (progress < 1.0) {
                    requestAnimationFrame(animateCutSequence);
                } else {
                    state.isTreeCutting = false;
                    btn.disabled = false;
                    btn.innerHTML = `<i class="fa-solid fa-scissors text-emerald-300 text-xl"></i> <span>この2点で丸太を切り出す！（チェンソー切断）</span>`;
                    btn.className = "w-full bg-gradient-to-r from-emerald-600 to-teal-800 hover:from-emerald-700 hover:to-teal-900 text-white font-bold py-4 px-6 rounded-xl shadow-lg btn-glow transition flex items-center justify-center gap-3 text-base";

                    if (cutHighlightMesh) {
                        cutHighlightMesh.material.color.setHex(0xf59e0b);
                    }

                    // 切断完了後にステップ②へ移行
                    goToStep(2);
                }
            };

            animateCutSequence();
        }

        function startCutAnimation() {
            if (state.isBoardCutting) return;
            state.isBoardCutting = true;
            state.cutProgress = 0;
            state.particles = [];

            const btn = document.getElementById('cutBtn');
            btn.disabled = true;
            btn.className = "w-full bg-slate-400 text-white font-bold py-3.5 px-6 rounded-xl cursor-not-allowed flex items-center justify-center gap-3 text-base";

            const info = getLineInfo();
            const duration = 1400;
            const startTime = Date.now();

            const animateSaw = () => {
                const now = Date.now();
                state.cutProgress = Math.min(1.0, (now - startTime) / duration);

                if (state.cutProgress < 1.0) {
                    const currentX = info.x1 + info.dx * state.cutProgress;
                    const currentY = info.y1 + info.dy * state.cutProgress;
                    for (let i = 0; i < 3; i++) {
                        state.particles.push({
                            x: currentX + (Math.random() - 0.5) * 15,
                            y: currentY + (Math.random() - 0.5) * 15,
                            vx: (Math.random() - 0.5) * 3,
                            vy: (Math.random() - 0.5) * 3,
                            size: Math.random() * 2.5 + 1,
                            color: Math.random() > 0.5 ? '#d49b6a' : '#f3dfc6',
                            life: 1.0
                        });
                    }
                }

                for (let p of state.particles) {
                    p.x += p.vx; p.y += p.vy; p.life -= 0.04;
                }
                state.particles = state.particles.filter(p => p.life > 0);

                updateAll();

                if (state.cutProgress < 1.0) {
                    requestAnimationFrame(animateSaw);
                } else {
                    state.isBoardCutting = false;
                    btn.disabled = false;
                    btn.className = "w-full bg-gradient-to-r from-amber-600 to-amber-800 hover:from-amber-700 hover:to-amber-900 text-white font-bold py-3.5 px-6 rounded-xl shadow-lg btn-glow transition flex items-center justify-center gap-3 text-base";
                    
                    setTimeout(() => goToStep(3), 200);
                }
            };
            animateSaw();
        }

        function toggleWarpAnimation() {
            const btn = document.getElementById('warpToggleBtn');
            const statusTag = document.getElementById('warpStatusTag');

            state.isWarped = !state.isWarped;
            if (state.isWarped) {
                btn.innerHTML = `<i class="fa-solid fa-rotate-left"></i> 未乾燥に戻す`;
                statusTag.innerText = "状態: 乾燥後（水分収縮により反り）";
            } else {
                btn.innerHTML = `<i class="fa-solid fa-wind"></i> 乾燥テスト（反り発生）`;
                statusTag.innerText = "状態: 未乾燥（直線の板）";
            }

            if (state.warpAnimId) cancelAnimationFrame(state.warpAnimId);

            const animate = () => {
                const target = state.isWarped ? 1 : 0;
                state.warpProgress += (target - state.warpProgress) * 0.1;

                const info = getLineInfo();
                apply3DWarp(info);

                if (Math.abs(target - state.warpProgress) > 0.01) {
                    state.warpAnimId = requestAnimationFrame(animate);
                } else {
                    state.warpProgress = target;
                    apply3DWarp(info);
                }
            };
            animate();
        }

        // ==========================================
        // 幾何計算 & 描画アップデート
        // ==========================================
        function getLineInfo() {
            const x1 = state.pointA.x, y1 = state.pointA.y;
            const x2 = state.pointB.x, y2 = state.pointB.y;

            const dx = x2 - x1, dy = y2 - y1;
            const len = Math.hypot(dx, dy);

            if (len === 0) return { dist: 0, angleRad: 0, isMasame: true };

            const c = x2 * y1 - x1 * y2;
            const signedDist = c / len;
            const absDist = Math.abs(signedDist);
            const angleRad = Math.atan2(dy, dx);
            const isMasame = absDist < (state.logRadius * 0.3);

            return { x1, y1, x2, y2, dx, dy, len, signedDist, absDist, angleRad, isMasame };
        }

        function updateAll() {
            const thickVal = document.getElementById('thickVal');
            if (thickVal) thickVal.innerText = `${state.thickness} mm`;

            const info = getLineInfo();

            const badge = document.getElementById('grainBadge');
            if (badge) {
                if (info.isMasame) {
                    badge.innerText = "柾目（まさめ）";
                    badge.className = "text-xs font-bold px-3 py-1 rounded-full bg-amber-100 text-amber-800 border border-amber-300";
                } else {
                    badge.innerText = "板目（いため）";
                    badge.className = "text-xs font-bold px-3 py-1 rounded-full bg-orange-100 text-orange-800 border border-orange-300";
                }
            }

            const predictBox = document.getElementById('grainPredictBox');
            if (predictBox) {
                if (info.isMasame) {
                    predictBox.className = "p-3 rounded-xl border border-amber-300 bg-amber-50 text-xs text-amber-900 space-y-1";
                    predictBox.innerHTML = `<span class="font-bold flex items-center gap-1"><i class="fa-solid fa-circle-check text-amber-700"></i> 切り出し予測：柾目（まさめ）</span><p class="text-[11px] text-amber-800">樹心（中心）近くを通る切断のため、年輪とほぼ直交し真っ直ぐな平行縞の木目が現れます。</p>`;
                } else {
                    predictBox.className = "p-3 rounded-xl border border-orange-300 bg-orange-50 text-xs text-orange-900 space-y-1";
                    predictBox.innerHTML = `<span class="font-bold flex items-center gap-1"><i class="fa-solid fa-circle-check text-orange-700"></i> 切り出し予測：板目（いため）</span><p class="text-[11px] text-orange-800">外皮に近い位置を切断するため、年輪が山形（筍目）に現れる自然な曲線模様になります。</p>`;
                }
            }

            if (state.currentStep === 2 && logCanvas) {
                drawLogCanvas(info);
            }

            updateSummary(info);
        }

        // 丸太断面 Canvas 描画
        function drawLogCanvas(info) {
            const rect = logCanvas.getBoundingClientRect();
            const w = rect.width, h = rect.height;
            const cx = w / 2, cy = h / 2;

            logCtx.clearRect(0, 0, w, h);
            logCtx.save();
            logCtx.translate(cx, cy);

            // 外皮
            logCtx.beginPath(); logCtx.arc(0, 0, state.logRadius, 0, Math.PI * 2);
            logCtx.fillStyle = "#8d5b4c"; logCtx.fill();

            // 辺材
            logCtx.beginPath(); logCtx.arc(0, 0, state.logRadius - 6, 0, Math.PI * 2);
            logCtx.fillStyle = "#f3dfc6"; logCtx.fill();

            // 心材
            const heartGrad = logCtx.createRadialGradient(0, 0, 0, 0, 0, state.logRadius * 0.65);
            heartGrad.addColorStop(0, '#d49b6a'); heartGrad.addColorStop(1, '#e2b388');
            logCtx.beginPath(); logCtx.arc(0, 0, state.logRadius * 0.65, 0, Math.PI * 2);
            logCtx.fillStyle = heartGrad; logCtx.fill();

            // 年輪
            logCtx.lineWidth = 1.5;
            for (let i = 1; i <= state.rings; i++) {
                const r = (state.logRadius - 8) * (i / state.rings);
                logCtx.beginPath(); logCtx.arc(0, 0, r, 0, Math.PI * 2);
                logCtx.strokeStyle = "rgba(120, 53, 15, 0.32)"; logCtx.stroke();
            }

            // 樹心
            logCtx.beginPath(); logCtx.arc(0, 0, 3.5, 0, Math.PI * 2);
            logCtx.fillStyle = "#5c2c16"; logCtx.fill();

            // 板の厚み可視化ガイド
            if (info.len > 0) {
                logCtx.save();
                const nx = -info.dy / info.len, ny = info.dx / info.len;
                const halfThick = state.thickness / 2;

                logCtx.strokeStyle = "rgba(194, 65, 12, 0.4)";
                logCtx.lineWidth = 1; logCtx.setLineDash([3, 3]);
                logCtx.beginPath();
                logCtx.moveTo(info.x1 + nx * halfThick, info.y1 + ny * halfThick);
                logCtx.lineTo(info.x2 + nx * halfThick, info.y2 + ny * halfThick);
                logCtx.moveTo(info.x1 - nx * halfThick, info.y1 - ny * halfThick);
                logCtx.lineTo(info.x2 - nx * halfThick, info.y2 - ny * halfThick);
                logCtx.stroke();
                logCtx.restore();
            }

            // 切断ライン
            logCtx.strokeStyle = "#ea580c"; logCtx.lineWidth = 3;
            logCtx.beginPath(); logCtx.moveTo(info.x1, info.y1); logCtx.lineTo(info.x2, info.y2); logCtx.stroke();

            // 木くず・のこぎり演出
            if (state.isBoardCutting && state.cutProgress < 1.0) {
                const currentX = info.x1 + info.dx * state.cutProgress;
                const currentY = info.y1 + info.dy * state.cutProgress;

                logCtx.save();
                logCtx.translate(currentX, currentY);
                logCtx.rotate(info.angleRad);
                const shake = Math.sin(Date.now() * 0.06) * 6;

                logCtx.fillStyle = "#cbd5e1"; logCtx.strokeStyle = "#334155"; logCtx.lineWidth = 1.5;
                logCtx.beginPath();
                logCtx.moveTo(-30 + shake, -5); logCtx.lineTo(26 + shake, -2);
                logCtx.lineTo(20 + shake, 12); logCtx.lineTo(-30 + shake, 5);
                logCtx.closePath(); logCtx.fill(); logCtx.stroke();

                logCtx.restore();

                for (let p of state.particles) {
                    logCtx.fillStyle = p.color;
                    logCtx.beginPath(); logCtx.arc(p.x, p.y, p.size, 0, Math.PI * 2); logCtx.fill();
                }
            }

            // 点A, 点B
            drawAnchorPoint(state.pointA.x, state.pointA.y, 'A', '#ef4444', state.activePoint === 'A');
            drawAnchorPoint(state.pointB.x, state.pointB.y, 'B', '#3b82f6', state.activePoint === 'B');

            logCtx.restore();
        }

        function drawAnchorPoint(x, y, label, color, isActive) {
            logCtx.save();
            logCtx.translate(x, y);

            logCtx.beginPath();
            logCtx.arc(0, 0, isActive ? 15 : 12, 0, Math.PI * 2);
            logCtx.fillStyle = color;
            logCtx.shadowColor = "rgba(0,0,0,0.3)";
            logCtx.shadowBlur = 5;
            logCtx.fill();

            logCtx.lineWidth = 2; logCtx.strokeStyle = "#ffffff"; logCtx.stroke();
            logCtx.fillStyle = "#ffffff"; logCtx.font = "bold 11px sans-serif";
            logCtx.textAlign = "center"; logCtx.textBaseline = "middle";
            logCtx.fillText(label, 0, 0.5);

            logCtx.restore();
        }

        function updateSummary(info) {
            const summaryBox = document.getElementById('featureSummary');
            if (!summaryBox) return;

            if (info.isMasame) {
                summaryBox.innerHTML = `
                    <div class="flex items-center gap-2 text-amber-300 font-bold text-sm">
                        <i class="fa-solid fa-star"></i> 切断面判定：柾目（まさめ）
                    </div>
                    <div class="grid grid-cols-1 gap-1.5 text-xs text-amber-100 pt-1">
                        <div>・<strong>木目模様：</strong> まっすぐな平行縦縞</div>
                        <div>・<strong>変形・反り：</strong> 非常に少なく狂いにくい</div>
                        <div>・<strong>主な用途：</strong> 高級家具、建具、楽器表板</div>
                        <div>・<strong>歩留まり：</strong> 丸太から少なく貴重</div>
                    </div>
                `;
            } else {
                summaryBox.innerHTML = `
                    <div class="flex items-center gap-2 text-orange-300 font-bold text-sm">
                        <i class="fa-solid fa-gem"></i> 切断面判定：板目（いため）
                    </div>
                    <div class="grid grid-cols-1 gap-1.5 text-orange-100 pt-1">
                        <div>・<strong>木目模様：</strong> 山形（筍目）の自然な曲線</div>
                        <div>・<strong>変形・反り：</strong> 乾燥すると木表側に反る</div>
                        <div>・<strong>主な用途：</strong> 建築一般材、DIY・木工用板</div>
                        <div>・<strong>歩留まり：</strong> 丸太から効率よく採れる</div>
                    </div>
                `;
            }
        }
    </script>
</body>
</html>
