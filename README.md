# TargetAnalyses--- START OF FILE Paste March 01, 2026 - 4:24PM ---

<!DOCTYPE html>
<html lang="en" dir="ltr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Soccer Analyst - Pro Recorder</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.sheetjs.com/xlsx-0.20.1/package/dist/xlsx.full.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700&display=swap" rel="stylesheet">
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: { sans: ['Cairo', 'sans-serif'] },
                    animation: { 
                        'pulse-fast': 'pulse 1.5s cubic-bezier(0.4, 0, 0.6, 1) infinite',
                        'fade-in': 'fadeIn 0.5s ease-out',
                        'flash-green': 'flashGreen 1.5s ease-out'
                    },
                    keyframes: {
                        fadeIn: { '0%': { opacity: '0' }, '100%': { opacity: '1' } },
                        flashGreen: { '0%': { backgroundColor: 'rgba(34, 197, 94, 0.5)' }, '100%': { backgroundColor: 'transparent' } }
                    }
                }
            }
        }
    </script>

    <style>
        .glass-panel { background: rgba(31, 41, 55, 0.95); backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.1); }
        .field-container { background: #1a1a1a; border: 4px solid #fff; position: relative; height: 550px; width: 100%; border-radius: 8px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.5); margin-top: 20px; }
        #tacticsCanvas { width: 100%; height: 100%; display: block; }
        .custom-scrollbar::-webkit-scrollbar { height: 8px; width: 8px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #1f2937; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #4b5563; border-radius: 4px; }
        .verdict-bar { transition: width 1s ease-in-out, background-color 0.5s; height: 100%; }
        #gameStopAlert { backdrop-filter: blur(10px); }
        .updated-flash { animation: flash-green 1s ease-out; }
        .rec-dot { height: 10px; width: 10px; background-color: #ef4444; border-radius: 50%; display: inline-block; animation: pulse-fast 1s infinite; }
        .field-preview { background-color: #2e8b57; border: 2px solid #fff; position: relative; height: 120px; width: 100%; border-radius: 4px; overflow: hidden; display: flex; justify-content: space-between; align-items: center; }
        .field-side { width: 50%; height: 100%; display: flex; justify-content: center; align-items: center; flex-direction: column; font-weight: bold; text-transform: uppercase; font-size: 0.8rem; text-shadow: 0 1px 2px rgba(0,0,0,0.8); transition: background 0.3s; }
        input[type="color"] { -webkit-appearance: none; border: none; width: 50px; height: 50px; border-radius: 10px; overflow: hidden; cursor: pointer; padding: 0; }
        input[type="color"]::-webkit-color-swatch-wrapper { padding: 0; }
        input[type="color"]::-webkit-color-swatch { border: none; border-radius: 10px; border: 2px solid rgba(255,255,255,0.2); }
    </style>
</head>
<body class="bg-gray-900 text-white min-h-screen font-sans transition-colors duration-300 relative overflow-x-hidden">

    <!-- SETUP WIZARD -->
    <div id="setupWizard" class="fixed inset-0 z-50 bg-gray-900 bg-[url('https://www.transparenttextures.com/patterns/cubes.png')] flex items-center justify-center p-4">
        <div id="step1" class="glass-panel max-w-2xl w-full rounded-2xl p-10 shadow-2xl text-center space-y-8">
            <h1 class="text-4xl font-bold text-green-400 mb-2">⚽ AI Soccer Analyst</h1>
            <p class="text-gray-300 text-lg">Select your footage source.</p>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mt-8">
                <button onclick="goToStep2('upload')" class="bg-gray-800 hover:bg-gray-700 border-2 border-gray-600 hover:border-green-500 rounded-xl p-8 transition-all transform hover:-translate-y-1">
                    <div class="text-5xl mb-4">📂</div><h3 class="text-xl font-bold">Upload Video</h3>
                </button>
                <button onclick="goToStep2('live')" class="bg-gray-800 hover:bg-gray-700 border-2 border-gray-600 hover:border-red-500 rounded-xl p-8 transition-all transform hover:-translate-y-1">
                    <div class="text-5xl mb-4">📡</div><h3 class="text-xl font-bold">Live Stream</h3>
                </button>
            </div>
        </div>

        <div id="step2" class="hidden glass-panel max-w-5xl w-full rounded-2xl p-8 shadow-2xl">
            <div class="flex justify-between items-center mb-6 border-b border-gray-700 pb-4">
                <h2 class="text-2xl font-bold flex items-center gap-2"><span class="bg-green-500 text-black w-8 h-8 rounded-full flex items-center justify-center text-sm">2</span> <span>Match Configuration</span></h2>
                <button onclick="resetToStep1()" class="text-gray-400 hover:text-white">✕ Cancel</button>
            </div>
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                <div class="lg:col-span-2 space-y-6">
                    <div id="sourceInputContainer">
                        <div id="wizUploadInput" class="hidden">
                            <label class="block text-sm font-bold text-gray-300 mb-2">Video File</label>
                            <input type="file" id="videoInputWiz" accept="video/*" class="block w-full text-sm text-gray-400 file:mr-4 file:py-2 file:px-4 file:rounded-full file:bg-gray-800 file:text-white border border-gray-600">
                        </div>
                        <div id="wizLiveInput" class="hidden">
                            <label class="block text-sm font-bold text-gray-300 mb-2">Stream URL (for reference)</label>
                            <input type="text" id="liveUrlWiz" placeholder="https://..." class="w-full bg-gray-800 border border-gray-600 rounded-lg px-4 py-3">
                        </div>
                    </div>
                    <div class="space-y-2">
                         <div class="flex justify-between items-center">
                            <label class="text-sm font-bold text-gray-300">Pitch Sides</label>
                            <button onclick="swapSides()" class="text-xs flex items-center gap-1 text-yellow-400 border border-yellow-400/30 px-2 py-1 rounded"><span>⇄</span> <span>Swap</span></button>
                        </div>
                        <div class="field-preview">
                            <div id="sideLeft" class="field-side" style="background: rgba(37, 99, 235, 0.3);"><span>Left</span><div class="mt-1 px-2 py-0.5 bg-black/50 rounded text-xs" id="labelLeft">Team A</div></div>
                            <div id="sideRight" class="field-side" style="background: rgba(220, 38, 38, 0.3);"><span>Right</span><div class="mt-1 px-2 py-0.5 bg-black/50 rounded text-xs" id="labelRight">Team B</div></div>
                        </div>
                    </div>
                    <div class="grid grid-cols-2 gap-4">
                        <div class="bg-gray-800 p-3 rounded-lg border border-gray-600 flex items-center gap-3">
                            <input type="color" id="wizColorA" value="#2563eb" onchange="updatePreviewColors()">
                            <div class="w-full"><span class="block font-bold text-sm" id="txtTeamA">Team A</span><input type="text" id="nameTeamA" class="bg-transparent border-b border-gray-600 text-xs w-full focus:outline-none" placeholder="Name..."></div>
                        </div>
                        <div class="bg-gray-800 p-3 rounded-lg border border-gray-600 flex items-center gap-3">
                            <input type="color" id="wizColorB" value="#dc2626" onchange="updatePreviewColors()">
                            <div class="w-full"><span class="block font-bold text-sm" id="txtTeamB">Team B</span><input type="text" id="nameTeamB" class="bg-transparent border-b border-gray-600 text-xs w-full focus:outline-none" placeholder="Name..."></div>
                        </div>
                    </div>
                </div>
                <div class="space-y-6 border-l border-gray-700 pl-6 lg:block">
                    <div>
                        <h3 class="font-bold text-green-400 mb-2"><span>📋</span> <span>Rosters (Excel)</span></h3>
                        <a href="#" onclick="downloadTemplate(event)" class="text-xs text-blue-400 underline mb-4 block">⬇ Download Template</a>
                    </div>
                    <div class="bg-gray-800 p-4 rounded-lg border border-gray-600 relative"><label class="block text-xs font-bold mb-1">Team A Roster</label><input type="file" accept=".xlsx, .csv" class="block w-full text-xs" onchange="handleRosterUpload(this, 'A')"><div id="statusA" class="absolute top-2 right-2 hidden text-green-500">✔</div></div>
                    <div class="bg-gray-800 p-4 rounded-lg border border-gray-600 relative"><label class="block text-xs font-bold mb-1">Team B Roster</label><input type="file" accept=".xlsx, .csv" class="block w-full text-xs" onchange="handleRosterUpload(this, 'B')"><div id="statusB" class="absolute top-2 right-2 hidden text-green-500">✔</div></div>
                    <div class="bg-gray-800/50 p-3 mt-4"><button onclick="simulateAutoDetect()" class="w-full bg-blue-600/20 text-blue-300 text-xs py-2 rounded"><span>🪄</span> <span>Auto-Detect</span></button></div>
                </div>
            </div>
            <div class="mt-8 pt-6 border-t border-gray-700 flex justify-end">
                <button onclick="finalizeSetup()" class="bg-green-600 hover:bg-green-500 text-white font-bold py-3 px-8 rounded-lg shadow-lg transform transition hover:scale-105"><span>▶</span> <span>RUN ANALYSIS</span></button>
            </div>
        </div>
    </div>

    <!-- MAIN DASHBOARD -->
    <div id="gameStopAlert" class="hidden fixed bottom-10 left-1/2 transform -translate-x-1/2 bg-red-600/90 border-2 border-red-400 text-white px-8 py-4 rounded-xl shadow-2xl z-50 animate-bounce-slow flex items-center gap-4">
        <span class="text-3xl bg-white rounded-full p-1">🛑</span>
        <div><strong class="block uppercase text-sm font-bold tracking-widest">ANALYSIS PAUSED</strong><span class="text-xs text-red-100">No motion detected.</span></div>
    </div>

    <header id="mainHeader" class="hidden bg-gray-800 p-6 shadow-lg border-b border-gray-700 sticky top-0 z-40">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-4">
                <h1 class="text-2xl font-bold text-green-400">⚽ <span data-i18n="headTitle">AI Soccer Analyst</span></h1>
                
                <!-- VIDEO TIME DISPLAY -->
                <div class="flex items-center gap-2 bg-black/40 border border-gray-600 px-4 py-1 rounded-full">
                    <span class="text-xs text-gray-400 uppercase font-bold" data-i18n="vidTime">VIDEO TIME</span>
                    <span id="matchClock" class="text-xl font-mono text-green-400 font-bold">00:00</span>
                </div>

                <div id="liveIndicator" class="hidden flex items-center gap-2 bg-red-600/20 border border-red-500 text-red-400 px-3 py-1 rounded-full animate-pulse-fast">
                    <span class="w-2 h-2 bg-red-500 rounded-full"></span><span class="text-xs font-bold uppercase">LIVE</span>
                </div>
            </div>
            <div id="fileLoadSection" class="hidden">
                <label class="cursor-pointer bg-blue-600 hover:bg-blue-500 text-white px-4 py-2 rounded-lg font-bold shadow-lg transition flex items-center gap-2">
                    <span>📂</span> <span data-i18n="btnLoad">Load Recorded File</span>
                    <input type="file" class="hidden" accept="video/*" onchange="loadRecordedFile(this)">
                </label>
            </div>
        </div>
    </header>

    <main class="max-w-7xl mx-auto p-6 relative">
        <div id="loadingSection" class="hidden py-32 text-center absolute inset-0 z-40 bg-gray-900 h-screen">
            <div class="inline-block loader mb-8"></div>
            <h2 class="text-3xl font-semibold animate-pulse text-green-400 mb-2">AI Analysis in Progress...</h2>
            <p id="loadingText" class="text-gray-400 text-xl font-mono">Initializing...</p>
        </div>

        <div id="dashboard" class="hidden space-y-8 mt-4 pb-20">

            <!-- LANGUAGE TOGGLE (TOP OF ANALYSES) -->
            <div class="flex justify-end items-center mb-4">
                <button id="langToggleBtn" onclick="toggleLanguage()" class="bg-gray-700 hover:bg-gray-600 border border-gray-500 text-green-400 font-bold py-2 px-6 rounded-full shadow-lg transition-all flex items-center gap-2">
                    <span>🌐</span> <span id="langLabel">English / العربية</span>
                </button>
            </div>
            
            <!-- DVR CONTROLS -->
            <div id="dvrContainer" class="bg-gray-800 border border-gray-700 p-6 rounded-xl shadow-lg animate-fade-in relative overflow-hidden">
                <div class="flex flex-col md:flex-row justify-between items-center gap-6">
                    <div class="flex-1">
                        <h3 class="text-lg font-bold text-white flex items-center gap-2">🔴 <span data-i18n="dvrTitle">Match Recorder (DVR)</span></h3>
                        <p class="text-gray-400 text-xs mt-1" data-i18n="dvrNote">
                            <span class="text-yellow-400 font-bold">NOTE:</span> When you click Start, your browser will ask 
                            "Choose what to share". Select the Tab playing the match.
                        </p>
                    </div>
                    <div class="flex items-center gap-4">
                        <div id="recStatus" class="hidden flex items-center gap-2 bg-black/40 px-3 py-1 rounded">
                            <span class="rec-dot"></span>
                            <span id="recTimer" class="font-mono text-red-400 font-bold">00:00</span>
                        </div>
                        <button id="btnStartRec" onclick="startDVR()" class="bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-6 rounded-lg shadow-lg transition flex items-center gap-2">
                            <span>🎥</span> <span data-i18n="btnStart">Start Recording</span>
                        </button>
                        <button id="btnStopRec" onclick="stopDVR()" class="hidden bg-gray-600 hover:bg-gray-500 text-white font-bold py-3 px-6 rounded-lg shadow-lg transition flex items-center gap-2">
                            <span>⏹</span> <span data-i18n="btnStop">Stop & Save</span>
                        </button>
                    </div>
                </div>
            </div>

            <!-- UPDATE TIMER -->
            <div id="timerSection" class="bg-gray-800 border-x-4 border-green-500 p-4 rounded-lg shadow-lg flex justify-between items-center max-w-xl mx-auto">
                <div>
                    <span class="block text-gray-400 text-xs uppercase font-bold tracking-widest" data-i18n="nextUpdate">Next AI Update (Progression):</span>
                    <div class="flex items-center gap-3 mt-1">
                        <span class="w-3 h-3 rounded-full bg-green-500 animate-pulse"></span>
                        <span id="countdownTimer" class="font-mono text-3xl font-bold text-white tracking-wider">05:00</span>
                    </div>
                </div>
                <div class="text-right">
                    <button onclick="forceUpdate()" class="text-xs bg-gray-700 hover:bg-gray-600 text-white px-2 py-1 rounded border border-gray-500" data-i18n="btnForce">Force +1 Min Update</button>
                    <span class="text-green-400 font-bold text-sm block mt-1">ACTIVE</span>
                </div>
            </div>

            <!-- Tactics Board -->
            <div class="bg-gray-800 rounded-xl p-6 shadow-xl border border-gray-700 relative">
                <div id="tacticsFlashOverlay" class="absolute inset-0 bg-green-500/0 pointer-events-none z-20 rounded-xl"></div>
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-6 gap-4">
                    <div><h2 class="text-2xl font-bold text-white" data-i18n="titleStrategy">Strategy Generator (4K)</h2><p class="text-gray-400 text-sm mt-1" data-i18n="titleSubStrategy">High-Precision AI Analysis</p></div>
                    <div class="flex flex-wrap gap-4 items-center">
                        <select id="teamSelector" onchange="updateTactics()" class="bg-gray-700 border border-gray-600 text-white rounded-lg px-4 py-2 shadow-sm"></select>
                        <div class="flex bg-gray-700 rounded-lg p-1 shadow-inner">
                            <button onclick="setPlan(1)" id="btnPlan1" class="px-4 py-2 rounded-md bg-green-600 text-white text-sm font-bold shadow">Smart Plan</button>
                            <button onclick="setPlan(2)" id="btnPlan2" class="px-4 py-2 rounded-md hover:bg-gray-600 text-gray-300 text-sm">Alt A</button>
                            <button onclick="setPlan(3)" id="btnPlan3" class="px-4 py-2 rounded-md hover:bg-gray-600 text-gray-300 text-sm">Alt B</button>
                            <button onclick="setPlan(4)" id="btnPlan4" class="px-4 py-2 rounded-md hover:bg-gray-600 text-gray-300 text-sm">Neutralize</button>
                            <button onclick="setPlan(5)" id="btnPlan5" class="px-4 py-2 rounded-md hover:bg-gray-600 text-gray-300 text-sm">Target</button>
                        </div>
                    </div>
                </div>
                <div id="planDescription" class="mb-4 p-5 bg-gray-700/50 rounded-lg border-l-4 border-green-500 text-gray-200 shadow-sm"></div>
                <div class="field-container">
                    <canvas id="tacticsCanvas"></canvas>
                </div>
            </div>

            <!-- Tables -->
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <!-- Team A -->
                <div class="bg-gray-800 rounded-xl shadow-xl overflow-hidden border border-gray-700 flex flex-col">
                    <div id="teamAHeader" class="p-4 flex justify-between items-center bg-gray-700">
                        <h2 class="text-xl font-bold flex items-center gap-2"><span id="teamABadge" class="text-xs font-bold px-2 py-1 rounded-full bg-white text-black">A</span><span id="lblTeamADash">Team A</span></h2>
                        <select id="sortA" onchange="handleSortChange('A')" class="bg-black/20 text-inherit border border-white/20 rounded px-2 py-1 text-xs"></select>
                    </div>
                    <div class="overflow-x-auto custom-scrollbar flex-grow">
                        <table class="w-full text-sm text-left"><thead class="bg-gray-900/50 text-gray-400"><tr><th class="py-3 px-4 w-16 text-center">#</th><th class="py-3 px-4 w-16 text-center">Img</th><th class="py-3 px-4" data-i18n="colPlayer">Name</th><th class="py-3 px-4 text-center" data-i18n="colSkill">Skill</th><th class="py-3 px-4 text-center" data-i18n="colSpeed">Speed</th><th class="py-3 px-4 text-end" data-i18n="colRating">Rating</th></tr></thead><tbody id="teamATableBody" class="divide-y divide-gray-700"></tbody></table>
                    </div>
                </div>
                <!-- Team B -->
                <div class="bg-gray-800 rounded-xl shadow-xl overflow-hidden border border-gray-700 flex flex-col">
                    <div id="teamBHeader" class="p-4 flex justify-between items-center bg-gray-700">
                        <h2 class="text-xl font-bold flex items-center gap-2"><span id="teamBBadge" class="text-xs font-bold px-2 py-1 rounded-full bg-white text-black">B</span><span id="lblTeamBDash">Team B</span></h2>
                        <select id="sortB" onchange="handleSortChange('B')" class="bg-black/20 text-inherit border border-white/20 rounded px-2 py-1 text-xs"></select>
                    </div>
                    <div class="overflow-x-auto custom-scrollbar flex-grow">
                        <table class="w-full text-sm text-left"><thead class="bg-gray-900/50 text-gray-400"><tr><th class="py-3 px-4 w-16 text-center">#</th><th class="py-3 px-4 w-16 text-center">Img</th><th class="py-3 px-4" data-i18n="colPlayer">Name</th><th class="py-3 px-4 text-center" data-i18n="colSkill">Skill</th><th class="py-3 px-4 text-center" data-i18n="colSpeed">Speed</th><th class="py-3 px-4 text-end" data-i18n="colRating">Rating</th></tr></thead><tbody id="teamBTableBody" class="divide-y divide-gray-700"></tbody></table>
                    </div>
                </div>
            </div>

            <!-- Detailed Analysis and Strategic Comments -->
            <div id="detailedAttacks" class="grid grid-cols-1 md:grid-cols-2 gap-8"></div>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
                 <!-- AI COACH INSIGHTS -->
                <div class="bg-gray-800 rounded-xl p-6 shadow-xl border border-gray-700">
                    <div class="border-b border-gray-700 pb-4 mb-4"><h3 class="text-xl font-bold text-yellow-400" data-i18n="titleCoach">🧠 AI Coach Insights</h3></div>
                    <div id="coachSuggestions" class="space-y-3"></div>
                </div>

                <!-- NEW: AI STRATEGIC COMMENTS (Subs & Passing Areas) -->
                <div class="md:col-span-2 bg-gray-800 rounded-xl p-6 shadow-xl border border-gray-700">
                    <div class="border-b border-gray-700 pb-4 mb-4">
                        <h3 class="text-xl font-bold text-blue-400" data-i18n="titleAdvancedAI">🚀 AI Tactical Deep-Dive</h3>
                    </div>
                    <div id="advancedStrategicComments" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <!-- Content generated by JS -->
                    </div>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="bg-gray-800 rounded-xl p-6 shadow-xl border border-gray-700 text-center">
                    <h3 class="text-lg font-bold text-gray-300 uppercase mb-4" data-i18n="titleVerdict">Verdict</h3>
                    <div class="relative h-6 bg-gray-700 rounded-full overflow-hidden w-full mb-4 flex">
                        <div id="barTeamA" class="verdict-bar flex items-center justify-start pl-2 text-xs font-bold text-white/90" style="width: 50%;"><span id="pctTeamA">50%</span></div>
                        <div id="barTeamB" class="verdict-bar flex items-center justify-end pr-2 text-xs font-bold text-white/90" style="width: 50%;"><span id="pctTeamB">50%</span></div>
                    </div>
                    <p id="verdictText" class="text-xl font-bold text-white"></p>
                </div>
            </div>

            <!-- TOTALS FOOTER -->
            <div class="fixed bottom-0 left-0 right-0 bg-gray-900 border-t border-gray-600 p-4 shadow-2xl z-50 flex justify-center gap-8 text-sm">
                <div class="flex items-center gap-4"><strong class="text-gray-400 uppercase" data-i18n="totalA">Team A Total:</strong><span class="text-blue-400 font-bold"><span data-i18n="colSkill">Skill</span>: <span id="ftSkillA">0</span></span><span class="text-yellow-400 font-bold"><span data-i18n="colSpeed">Speed</span>: <span id="ftSpeedA">0</span></span></div>
                <div class="w-px bg-gray-700 h-6"></div>
                <div class="flex items-center gap-4"><strong class="text-gray-400 uppercase" data-i18n="totalB">Team B Total:</strong><span class="text-red-400 font-bold"><span data-i18n="colSkill">Skill</span>: <span id="ftSkillB">0</span></span><span class="text-yellow-400 font-bold"><span data-i18n="colSpeed">Speed</span>: <span id="ftSpeedB">0</span></span></div>
            </div>
        </div>
    </main>

    <script>
        // --- CONFIG ---
        let currentLang='en', teamColors={}, teamA=[], teamB=[], rosterDataA={}, rosterDataB={}, currentSort={A:'rating',B:'rating'}, generatedTactics={A:[],B:[]}, currentPlan=1, selectedMode='upload', isTeamASwapped=false, liveIntervalId=null, animFrameId=null;
        
        let liveState = { isActive: false, reportIntervalMinutes: 5, reportTimerSeconds: 300, motionLevel: 100 };
        let matchMinutes = 0; 
        
        let mediaRecorder, recordedChunks=[], dvrInterval, dvrSeconds=0, recordingVersion=1, RECORDING_LIMIT=600;

        const dictionary = { 
            en: { 
                headTitle: "AI Soccer Analyst", vidTime: "VIDEO TIME", btnLoad: "Load Recorded File",
                dvrTitle: "Match Recorder (DVR)", dvrNote: "NOTE: When you click Start, your browser will ask \"Choose what to share\". Select the Tab playing the match.",
                btnStart: "Start Recording", btnStop: "Stop & Save", nextUpdate: "Next AI Update:", btnForce: "Force Update",
                titleStrategy: "Strategy Generator (4K)", titleSubStrategy: "High-Precision AI Analysis",
                totalA: "Team A Total", totalB: "Team B Total",
                total: "TOTALS", selectOptA: "Team A Attack", selectOptB: "Team B Attack", 
                atkTitle: "Key Attack Opportunities", spdAdv: "Speed Mismatch", sklAdv: "Technical Dribble", tacAdv: "Tactical Overload",
                colRating: "Rating", colSkill: "Skill", colSpeed: "Speed", colPlayer: "Name",
                titleCoach: "🧠 AI Coach Insights", titleVerdict: "Verdict",
                titleAdvancedAI: "🚀 AI Tactical Deep-Dive", labelSub: "Suggested Substitutions", labelPass: "Target Passing Lane",
                dangerousPlayers: "Top Dangerous Players", optimalFormation: "Best Strategic Formation"
            }, 
            ar: { 
                headTitle: "محلل كرة القدم الذكي", vidTime: "وقت الفيديو", btnLoad: "تحميل ملف مسجل",
                dvrTitle: "مسجل المباراة (DVR)", dvrNote: "ملاحظة: عند النقر على بدء، سيطلب متصفحك تحديد النافذة. اختر علامة التبويب التي تشغل المباراة.",
                btnStart: "بدء التسجيل", btnStop: "إيقاف وحفظ", nextUpdate: "تحديث الذكاء الاصطناعي القادم:", btnForce: "تحديث إجباري",
                titleStrategy: "مولد الاستراتيجية (4K)", titleSubStrategy: "تحليل عالي الدقة بالذكاء الاصطناعي",
                totalA: "مجموع الفريق أ", totalB: "مجموع الفريق ب",
                total: "المجموع", selectOptA: "هجوم فريق أ", selectOptB: "هجوم فريق ب", 
                atkTitle: "فرص الهجوم الرئيسية", spdAdv: "تفوق سرعة", sklAdv: "مهارة مراوغة", tacAdv: "زيادة عددية",
                colRating: "تقييم", colSkill: "مهارة", colSpeed: "سرعة", colPlayer: "الاسم",
                titleCoach: "🧠 رؤى المدرب الذكي", titleVerdict: "النتيجة النهائية",
                titleAdvancedAI: "🚀 تعمق تكتيكي ذكي", labelSub: "قائمة التبديلات المطلوبة", labelPass: "منطقة التمرير المستهدفة",
                dangerousPlayers: "أخطر اللاعبين في الفريق", optimalFormation: "أفضل تشكيلة لمواجهة الخصم"
            } 
        };

        function downloadTemplate(e) { e.preventDefault(); const ws=XLSX.utils.aoa_to_sheet([["Number","Name","Position"],[1,"GK Name","GK"],[10,"Striker Name","FWD"]]); const wb=XLSX.utils.book_new(); XLSX.utils.book_append_sheet(wb,ws,"Template"); XLSX.writeFile(wb,"Roster_Template.xlsx"); }
        
        // --- UPDATED ROSTER UPLOAD ---
        function handleRosterUpload(input,tp){ 
            const f=input.files[0]; if(!f)return; 
            const r=new FileReader(); 
            r.onload=e=>{ 
                const d=new Uint8Array(e.target.result), wb=XLSX.read(d,{type:'array'}), j=XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]],{header:1}), m={}; 
                for(let i=1;i<j.length;i++){
                    const row=j[i]; 
                    if(row && row.length>=2) {
                        const pNum = parseInt(row[0]);
                        if(!isNaN(pNum)) {
                            m[pNum] = { name: row[1].toString(), role: (row[2] || "MID").toString() };
                        }
                    }
                } 
                if(tp==='A'){ rosterDataA=m; document.getElementById('statusA').classList.remove('hidden'); }
                else { rosterDataB=m; document.getElementById('statusB').classList.remove('hidden'); } 
            }; 
            r.readAsArrayBuffer(f); 
        }

        function goToStep2(m){selectedMode=m;document.getElementById('step1').classList.add('hidden');document.getElementById('step2').classList.remove('hidden');document.getElementById(m==='upload'?'wizUploadInput':'wizLiveInput').classList.remove('hidden');document.getElementById(m==='upload'?'wizLiveInput':'wizUploadInput').classList.add('hidden');updatePreviewColors();}
        function resetToStep1(){document.getElementById('step2').classList.add('hidden');document.getElementById('step1').classList.remove('hidden');}
        function swapSides(){isTeamASwapped=!isTeamASwapped;document.getElementById('labelLeft').innerText=isTeamASwapped?"Team B":"Team A";document.getElementById('labelRight').innerText=isTeamASwapped?"Team A":"Team B";updatePreviewColors();}
        function updatePreviewColors(){const cA=document.getElementById('wizColorA').value,cB=document.getElementById('wizColorB').value,l=document.getElementById('sideLeft'),r=document.getElementById('sideRight');const h2r=(h,a)=>`rgba(${parseInt(h.slice(1,3),16)},${parseInt(h.slice(3,5),16)},${parseInt(h.slice(5,7),16)},${a})`;if(isTeamASwapped){l.style.background=h2r(cB,0.4);r.style.background=h2r(cA,0.4);}else{l.style.background=h2r(cA,0.4);r.style.background=h2r(cB,0.4);}}
        function simulateAutoDetect(){const c=["#e11d48","#2563eb","#facc15","#16a34a","#ffffff","#000000"];document.getElementById('wizColorA').value=c[Math.floor(Math.random()*6)];document.getElementById('wizColorB').value=c[Math.floor(Math.random()*6)];updatePreviewColors();alert("Colors detected!");}
        
        function finalizeSetup(){
            if(selectedMode==='upload'&&!document.getElementById('videoInputWiz').files[0])return alert("Please select a video file.");
            if(selectedMode==='live'&&!document.getElementById('liveUrlWiz').value)return alert("Please enter a Stream URL.");
            const nA=document.getElementById('nameTeamA').value||"Team A",nB=document.getElementById('nameTeamB').value||"Team B";
            document.getElementById('lblTeamADash').innerText=nA;
            document.getElementById('lblTeamBDash').innerText=nB;
            document.getElementById('setupWizard').classList.add('hidden');
            document.getElementById('mainHeader').classList.remove('hidden');
            handleProcessing(selectedMode);
        }

        function resolvePlayerImage(n){return `https://ui-avatars.com/api/?name=${encodeURIComponent(n)}&background=random&color=fff&size=128`;}
        
        function handleProcessing(s){
            liveState.isActive=true; liveState.reportTimerSeconds=300; matchMinutes = 0; 
            const getC=(h)=>(((parseInt(h.substr(1,2),16)*299)+(parseInt(h.substr(3,2),16)*587)+(parseInt(h.substr(5,2),16)*114))/1000)>=128?'text-gray-900':'text-white';
            teamColors={A:{hex:document.getElementById('wizColorA').value,textClass:getC(document.getElementById('wizColorA').value)},B:{hex:document.getElementById('wizColorB').value,textClass:getC(document.getElementById('wizColorB').value)}};
            document.getElementById('loadingSection').classList.remove('hidden');
            if(s === 'live') { document.getElementById('dvrContainer').classList.remove('hidden'); } else { document.getElementById('dvrContainer').classList.add('hidden'); }
            
            teamA=genData("A",rosterDataA); 
            teamB=genData("B",rosterDataB); 
            
            genTactics();
            setTimeout(()=>{document.getElementById('loadingSection').classList.add('hidden');document.getElementById('dashboard').classList.remove('hidden');applyColors();updDrops();renderT();initCanvas();currentPlan=1;updateTactics();renderCoach();renderAttackScenarios();renderPerf();renderStrategicComments();startTimer();},2500);
        }

        function genData(prefix, rosterMap){
            let arr=[];
            let excelNums = Object.keys(rosterMap).map(n => parseInt(n));
            let finalNums = [...new Set([...excelNums, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11])].sort((a,b)=>a-b).slice(0,11);
            finalNums.forEach(num => {
                let finalName = `${prefix === 'A' ? 'Player A' : 'Player B'} ${num}`;
                let finalRole = (num === 1 ? 'GK' : (num < 6 ? 'DEF' : (num < 10 ? 'MID' : 'FWD')));
                if(rosterMap && rosterMap[num]) {
                    finalName = rosterMap[num].name;
                    finalRole = rosterMap[num].role;
                }
                arr.push(mkP(num, prefix, finalName, finalRole));
            });
            return arr;
        }

        function mkP(n,p,nm,r){
            const s=Math.floor(Math.random()*(95-65)+65),
            sp=(Math.random()*(36-25)+25).toFixed(1);
            return{
                number:n,
                role:r,
                teamPrefix:p,
                skill:s,
                speed:sp,
                score:(s+parseFloat(sp)).toFixed(1),
                customName:nm,
                imageUrl:resolvePlayerImage(nm)
            };
        }

        function applyColors(){const u=(id,c,b)=>{const e=document.getElementById(id);e.style.backgroundColor=c.hex;e.className=`p-4 flex justify-between items-center ${c.textClass}`;document.getElementById(b).style.color=c.hex;document.getElementById(b).style.backgroundColor=c.textClass.includes('white')?'white':'black';};u('teamAHeader',teamColors.A,'teamABadge');u('teamBHeader',teamColors.B,'teamBBadge');}

        async function startDVR() {
            try {
                if(!confirm("⚠️ BROWSER SECURITY NOTICE:\n\nYou must select the WINDOW or TAB playing the match in the next popup.\n\nThis is a requirement to record your screen.\nClick OK to select your screen.")) return;
                const stream = await navigator.mediaDevices.getDisplayMedia({ video: true, audio: true });
                const options = { mimeType: MediaRecorder.isTypeSupported('video/mp4') ? 'video/mp4' : 'video/webm' };
                mediaRecorder = new MediaRecorder(stream, options);
                recordedChunks = [];
                mediaRecorder.ondataavailable = e => { if(e.data.size > 0) recordedChunks.push(e.data); };
                mediaRecorder.onstop = saveRecording;
                mediaRecorder.start();
                document.getElementById('btnStartRec').classList.add('hidden');
                document.getElementById('btnStopRec').classList.remove('hidden');
                document.getElementById('recStatus').classList.remove('hidden');
                document.getElementById('recStatus').classList.add('flex');
                dvrSeconds = 0;
                dvrInterval = setInterval(() => {
                    dvrSeconds++;
                    const m = Math.floor(dvrSeconds/60).toString().padStart(2,'0'), s = (dvrSeconds%60).toString().padStart(2,'0');
                    document.getElementById('recTimer').innerText = `${m}:${s}`;
                    if(dvrSeconds >= RECORDING_LIMIT) stopDVR();
                }, 1000);
            } catch(err) { console.error(err); alert("Recording cancelled by user."); }
        }
        function stopDVR() {
            if(mediaRecorder && mediaRecorder.state !== 'inactive') mediaRecorder.stop();
            clearInterval(dvrInterval);
            document.getElementById('btnStartRec').classList.remove('hidden');
            document.getElementById('btnStopRec').classList.add('hidden');
            document.getElementById('recStatus').classList.add('hidden');
            document.getElementById('recStatus').classList.remove('flex');
            mediaRecorder.stream.getTracks().forEach(t => t.stop());
        }
        function saveRecording() {
            if (recordedChunks.length === 0) return;
            const blob = new Blob(recordedChunks, { type: 'video/mp4' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            let filename = `Match_Rec_v${recordingVersion}.mp4`;
            recordingVersion++;
            a.style.display = 'none'; a.href = url; a.download = filename;
            document.body.appendChild(a); a.click();
            setTimeout(() => { document.body.removeChild(a); window.URL.revokeObjectURL(url); }, 100);
            document.getElementById('fileLoadSection').classList.remove('hidden');
        }
        function loadRecordedFile(input) { if(input.files && input.files[0]) { handleProcessing('upload'); } }

        function renderT(){
            const row=(p,c)=>`<tr class="updated-flash border-b border-gray-700 hover:bg-gray-700/50"><td class="py-3 px-4 text-center"><span class="inline-block w-8 h-8 leading-8 rounded-full font-bold text-xs" style="background:${c.hex};color:${c.textClass.includes('white')?'white':'black'}">${p.number}</span></td><td class="text-center"><img src="${p.imageUrl}" class="w-10 h-10 rounded-full mx-auto"></td><td class="font-medium px-4">${p.customName}<span class="block text-xs text-gray-500">${p.role}</span></td><td class="text-center text-gray-300 font-bold">${p.skill}</td><td class="text-center text-yellow-400 font-mono font-bold">${p.speed}</td><td class="text-end font-bold text-green-400 px-4">${p.score}</td></tr>`;
            const sFn=(l,c)=>[...l].sort((a,b)=>c==='skill'?b.skill-a.skill:c==='speed'?b.speed-a.speed:b.score-a.score);
            document.getElementById('teamATableBody').innerHTML=sFn(teamA,currentSort.A).map(p=>row(p,teamColors.A)).join('');
            document.getElementById('teamBTableBody').innerHTML=sFn(teamB,currentSort.B).map(p=>row(p,teamColors.B)).join('');
            const tot=(a)=>a.reduce((x,y)=>({skill:x.skill+y.skill,speed:x.speed+parseFloat(y.speed)}),{skill:0,speed:0});
            const tA=tot(teamA), tB=tot(teamB);
            document.getElementById('ftSkillA').innerText=tA.skill; document.getElementById('ftSpeedA').innerText=tA.speed.toFixed(1);
            document.getElementById('ftSkillB').innerText=tB.skill; document.getElementById('ftSpeedB').innerText=tB.speed.toFixed(1);
        }

        function renderAttackScenarios() {
            const container = document.getElementById('detailedAttacks'); container.innerHTML = "";
            const nA = document.getElementById('lblTeamADash').innerText, nB = document.getElementById('lblTeamBDash').innerText;
            const gen = (atk, def, nm) => {
                let h = `<div class="bg-gray-800 rounded-xl p-5 border border-gray-700"><h4 class="text-green-400 font-bold mb-3 uppercase tracking-wide border-b border-gray-700 pb-2">${nm} ${dictionary[currentLang].atkTitle}</h4><div class="space-y-3">`;
                const a = atk.filter(p=>['FWD','MID'].includes(p.role)).sort((a,b)=>b.speed-a.speed);
                const d = def.filter(p=>['DEF','GK'].includes(p.role)).sort((a,b)=>a.speed-b.speed);
                if(a.length && d.length && (parseFloat(a[0].speed)>parseFloat(d[0].speed)+2)) h+=`<div class="flex items-center gap-3 bg-gray-900/50 p-2 rounded border-l-2 border-yellow-500"><span class="text-xl">⚡</span><div><strong class="text-sm text-yellow-100">${dictionary[currentLang].spdAdv}</strong><p class="text-xs text-gray-400">${a[0].customName} > ${d[0].customName}</p></div></div>`;
                const tm = atk.filter(p=>p.role==='MID').sort((a,b)=>b.skill-a.skill);
                const wd = def.filter(p=>p.role==='DEF').sort((a,b)=>a.skill-b.skill);
                if(tm.length && wd.length && (tm[0].skill>wd[0].skill+8)) h+=`<div class="flex items-center gap-3 bg-gray-900/50 p-2 rounded border-l-2 border-purple-500"><span class="text-xl">🪄</span><div><strong class="text-sm text-purple-100">${dictionary[currentLang].sklAdv}</strong><p class="text-xs text-gray-400">${tm[0].customName} vs ${wd[0].customName}</p></div></div>`;
                h += `</div></div>`; return h;
            };
            container.innerHTML += gen(teamA, teamB, nA); container.innerHTML += gen(teamB, teamA, nB);
        }

        // --- NEW STRATEGIC COMMENTS LOGIC ---
        function renderStrategicComments() {
            const container = document.getElementById('advancedStrategicComments');
            container.innerHTML = "";

            const nA = document.getElementById('lblTeamADash').innerText;
            const nB = document.getElementById('lblTeamBDash').innerText;

            const analyzeTeam = (myTeam, oppTeam, teamName, isArabic) => {
                // Top 3 Dangerous Players
                const dangerous = [...myTeam].sort((a,b) => b.score - a.score).slice(0,3);
                
                // Players to be replaced (Bottom 2 by score)
                const replacements = [...myTeam].sort((a,b) => a.score - b.score).slice(0,2);
                
                // Formation Logic
                const avgSpeed = myTeam.reduce((acc, p) => acc + parseFloat(p.speed), 0) / myTeam.length;
                const avgSkill = myTeam.reduce((acc, p) => acc + p.skill, 0) / myTeam.length;
                let formationText = "4-4-2 Balanced";
                if(avgSpeed > 31) formationText = "4-3-3 Counter-Attack";
                else if(avgSkill > 80) formationText = "4-2-3-1 Possession";

                const oppWeakLink = [...oppTeam].filter(p => ['DEF','MID'].includes(p.role)).sort((a,b) => a.score - b.score)[0];
                let sideLabel = oppWeakLink.number % 2 === 0 ? (isArabic ? "الجناح الأيمن" : "Right Wing") : (isArabic ? "الجناح الأيسر" : "Left Wing");
                
                const passTxt = isArabic
                    ? `استهدف <b>${sideLabel}</b> لتجاوز الخصم <b>#${oppWeakLink.number}</b>.`
                    : `Target the <b>${sideLabel}</b> to exploit opponent <b>#${oppWeakLink.number}</b>.`;

                return `
                    <div class="bg-gray-900/40 p-4 rounded-lg border border-gray-700">
                        <h4 class="font-bold text-white mb-3 flex items-center gap-2">
                             <span class="w-3 h-3 rounded-full" style="background:${teamColors[myTeam[0].teamPrefix].hex}"></span>
                             ${teamName}
                        </h4>
                        <div class="space-y-4">
                            <div class="text-xs">
                                <span class="text-yellow-400 font-bold block mb-1 uppercase">🔥 ${dictionary[currentLang].dangerousPlayers}</span>
                                <ul class="list-disc list-inside text-gray-300">
                                    ${dangerous.map(p => `<li>#${p.number} ${p.customName} (${p.score})</li>`).join('')}
                                </ul>
                            </div>
                            <div class="text-xs">
                                <span class="text-red-400 font-bold block mb-1 uppercase">🔄 ${dictionary[currentLang].labelSub}</span>
                                <ul class="list-disc list-inside text-gray-300">
                                    ${replacements.map(p => `<li>#${p.number} ${p.customName}</li>`).join('')}
                                </ul>
                            </div>
                            <div class="text-xs">
                                <span class="text-blue-400 font-bold block mb-1 uppercase">📋 ${dictionary[currentLang].optimalFormation}</span>
                                <p class="text-gray-300 font-bold">${formationText}</p>
                            </div>
                            <div class="text-xs">
                                <span class="text-green-400 font-bold block mb-1 uppercase">🎯 ${dictionary[currentLang].labelPass}</span>
                                <p class="text-gray-300">${passTxt}</p>
                            </div>
                        </div>
                    </div>
                `;
            };

            const isAr = currentLang === 'ar';
            container.innerHTML += analyzeTeam(teamA, teamB, nA, isAr);
            container.innerHTML += analyzeTeam(teamB, teamA, nB, isAr);
        }

        function startTimer(){ if(liveIntervalId)clearInterval(liveIntervalId); liveIntervalId=setInterval(()=>{liveState.reportTimerSeconds--; if(liveState.reportTimerSeconds<=0){triggerUpdate();} const m=Math.floor(liveState.reportTimerSeconds/60).toString().padStart(2,'0'), s=(liveState.reportTimerSeconds%60).toString().padStart(2,'0'); document.getElementById('countdownTimer').innerText=`${m}:${s}`;},1000);}
        function forceUpdate(){liveState.reportTimerSeconds=0;triggerUpdate();}
        
        function triggerUpdate(){
            document.getElementById('countdownTimer').innerText="UPDATING..."; document.getElementById('countdownTimer').classList.add('text-yellow-400'); document.getElementById('tacticsFlashOverlay').classList.add('updated-flash');
            matchMinutes += 1;
            document.getElementById('matchClock').innerText = `${matchMinutes.toString().padStart(2,'0')}:00`;
            [teamA,teamB].forEach(t=>t.forEach(p=>{let d=(Math.random()*4)-2;p.skill=Math.max(50,Math.min(99,Math.floor(p.skill+d)));p.score=(p.skill+parseFloat(p.speed)).toFixed(1);}));
            genTactics(); setTimeout(()=>{ updateTactics(); renderT(); renderCoach(); renderAttackScenarios(); renderPerf(); renderStrategicComments(); liveState.reportTimerSeconds=300; document.getElementById('countdownTimer').classList.remove('text-yellow-400'); document.getElementById('tacticsFlashOverlay').classList.remove('updated-flash'); },1000);
        }

        function genTactics(){ 
            ['A','B'].forEach(tId=>{
                generatedTactics[tId]=[];
                const mine = (tId === 'A') ? teamA : teamB;
                const opp = (tId === 'A') ? teamB : teamA;
                const rankMine = [...mine].sort((a,b)=>b.score - a.score);
                const rankOpp = [...opp].sort((a,b)=>b.score - a.score);

                if (tId === 'A') {
                    generatedTactics[tId].push({ p: [1, 5, 8], d: `<b>Aggressive Push:</b> Strategy relies on performance leaders <b>#${rankMine[0].number}</b> and <b>#${rankMine[1].number}</b> pushing the center line, supported by <b>#${rankMine[2].number}</b>.` });
                    generatedTactics[tId].push({ p: [3, 4, 9], d: `<b>Wing Rotation:</b> Utilize the stamina of <b>#${rankMine[0].number}</b> to create space on the left, feeding <b>#${rankMine[1].number}</b> in the box.` });
                    generatedTactics[tId].push({ p: [0, 2, 7], d: `<b>Deep Build-up:</b> Reliable ball retention from <b>#${rankMine[0].number}</b> and <b>#${rankMine[3].number}</b> to draw the defense out.` });
                    generatedTactics[tId].push({ p: [2, 5, 1], d: `<b>Neutralize Opponent:</b> <b>#${rankMine[1].number}</b> is assigned to shadow opponent's top threat <b>#${rankOpp[0].number}</b>.` });
                    generatedTactics[tId].push({ p: [3, 9, 8], d: `<b>Clinical Strike:</b> Feed the ball directly to <b>#${rankMine[0].number}</b> to exploit the low defensive rating of opponent <b>#${rankOpp[10] ? rankOpp[10].number : '?'}</b>.` });
                } else {
                    generatedTactics[tId].push({ p: [2, 6, 7], d: `<b>Counter Strike:</b> Rapid transition plan led by speed leader <b>#${rankMine[0].number}</b> and playmaker <b>#${rankMine[2].number}</b>.` });
                    generatedTactics[tId].push({ p: [1, 4, 10], d: `<b>High Press:</b> <b>#${rankMine[0].number}</b>, <b>#${rankMine[1].number}</b>, and <b>#${rankMine[2].number}</b> to squeeze the opponent's goalkeeper distribution.` });
                    generatedTactics[tId].push({ p: [0, 5, 8], d: `<b>Central Overload:</b> <b>#${rankMine[0].number}</b> and <b>#${rankMine[4].number}</b> to dominate the midfield circle and control the tempo.` });
                    generatedTactics[tId].push({ p: [4, 5, 1], d: `<b>Defensive Lock:</b> Double team the opponent <b>#${rankOpp[0].number}</b> using <b>#${rankMine[3].number}</b> as the primary anchor.` });
                    generatedTactics[tId].push({ p: [7, 8, 9], d: `<b>Final Third Blitz:</b> Performance leaders <b>#${rankMine[0].number}</b> and <b>#${rankMine[1].number}</b> to execute a 1-2 pass sequence in the box.` });
                }
            }); 
        }

        const formation=[{x:0.05,y:0.5},{x:0.20,y:0.2},{x:0.20,y:0.5},{x:0.20,y:0.8},{x:0.40,y:0.3},{x:0.40,y:0.5},{x:0.40,y:0.7},{x:0.65,y:0.2},{x:0.65,y:0.5},{x:0.65,y:0.8},{x:0.2,y:0.65}];
        let ctx, canvasWidth, canvasHeight, lineDashOffset=0;
        function initCanvas(){ const c=document.getElementById('tacticsCanvas'), cont=c.parentElement, dpr=window.devicePixelRatio||1, rect=cont.getBoundingClientRect(); c.width=rect.width*dpr; c.height=rect.height*dpr; ctx=c.getContext('2d'); ctx.scale(dpr,dpr); canvasWidth=rect.width; canvasHeight=rect.height; if(!animFrameId) animLoop(); }
        function animLoop(){ lineDashOffset-=0.5; if(ctx) drawScene(); animFrameId=requestAnimationFrame(animLoop); }
        function setPlan(n){ currentPlan=n; [1,2,3,4,5].forEach(i=>{const b=document.getElementById(`btnPlan${i}`); if(!b) return; if(i===n){ b.classList.add('bg-green-600','text-white'); b.classList.remove('text-gray-300'); } else{ b.classList.remove('bg-green-600','text-white'); b.classList.add('text-gray-300'); }}); updateTactics(); }
        function updateTactics(){ 
            const t=document.getElementById('teamSelector').value;
            if(!generatedTactics[t]) return;
            const p=generatedTactics[t][currentPlan-1]; 
            document.getElementById('planDescription').innerHTML=`<div class="flex items-start gap-3"><span class="text-2xl">🎯</span><div><strong class="block text-lg text-green-400">Plan ${currentPlan}</strong><p class="text-gray-300 text-sm">${p.d}</p></div></div>`; 
        }
        function drawScene(){
            if(!ctx)return; ctx.clearRect(0,0,canvasWidth,canvasHeight);
            const stripeH=canvasHeight/10; for(let i=0;i<10;i++){ctx.fillStyle=i%2===0?'#2e8b57':'#3cb371';ctx.fillRect(0,i*stripeH,canvasWidth,stripeH);}
            ctx.strokeStyle='rgba(255,255,255,0.7)'; ctx.lineWidth=2; ctx.strokeRect(20,20,canvasWidth-40,canvasHeight-40); ctx.beginPath(); ctx.moveTo(canvasWidth/2,20); ctx.lineTo(canvasWidth/2,canvasHeight-20); ctx.stroke(); ctx.beginPath(); ctx.arc(canvasWidth/2,canvasHeight/2,50,0,Math.PI*2); ctx.stroke();
            const tId=document.getElementById('teamSelector').value||'A', plan=generatedTactics[tId]?generatedTactics[tId][currentPlan-1]:null, teamArr=tId==='A'?teamA:teamB, color=tId==='A'?teamColors.A.hex:teamColors.B.hex, isL2R=tId==='A'?!isTeamASwapped:isTeamASwapped;
            if(plan&&plan.p){ const pts=plan.p.map(i=>{const fx=isL2R?formation[i].x:1-formation[i].x;return{x:fx*canvasWidth,y:formation[i].y*canvasHeight};}); if(pts.length>0){ ctx.beginPath(); ctx.lineWidth=6; ctx.strokeStyle='#fbbf24'; ctx.shadowBlur=15; ctx.shadowColor='#fbbf24'; ctx.setLineDash([15,10]); ctx.lineDashOffset=lineDashOffset; ctx.moveTo(pts[0].x,pts[0].y); for(let i=1;i<pts.length;i++) ctx.lineTo(pts[i].x,pts[i].y); const gx=isL2R?canvasWidth-20:20; ctx.lineTo(gx,canvasHeight/2); ctx.stroke(); ctx.setLineDash([]); ctx.lineDashOffset=0; ctx.shadowBlur=0; ctx.beginPath(); ctx.fillStyle='#fbbf24'; const dir=isL2R?-1:1; ctx.moveTo(gx,canvasHeight/2); ctx.lineTo(gx+15*dir,canvasHeight/2-10); ctx.lineTo(gx+15*dir,canvasHeight/2+10); ctx.fill(); } }
            teamArr.forEach((p,i)=>{ if(i>=formation.length)return; const fx=isL2R?formation[i].x:1-formation[i].x, px=fx*canvasWidth, py=formation[i].y*canvasHeight; ctx.beginPath(); ctx.arc(px,py+4,14,0,Math.PI*2); ctx.fillStyle='rgba(0,0,0,0.5)'; ctx.fill(); const pGrad=ctx.createRadialGradient(px-5,py-5,2,px,py,14); pGrad.addColorStop(0,lighten(color,20)); pGrad.addColorStop(1,color); ctx.beginPath(); ctx.arc(px,py,14,0,Math.PI*2); ctx.fillStyle=pGrad; ctx.fill(); ctx.strokeStyle='#fff'; ctx.lineWidth=2; ctx.stroke(); ctx.fillStyle='#fff'; ctx.font='bold 11px Arial'; ctx.textAlign='center'; ctx.textBaseline='middle'; ctx.fillText(p.number,px,py); });
        }
        function lighten(h,a){let n=parseInt(h.slice(1),16),r=(n>>16)+a,b=((n>>8)&0x00FF)+a,g=(n&0x0000FF)+a;const x=(v)=>Math.min(255,Math.max(0,v));return `rgb(${x(r)},${x(b)},${x(g)})`;}
        function renderCoach(){ const c=document.getElementById('coachSuggestions'); c.innerHTML=""; const logic=(a,d)=>{const at=a.filter(p=>['FWD','MID'].includes(p.role)).sort((x,y)=>y.speed-x.speed).slice(0,2), df=d.filter(p=>p.role==='DEF').sort((x,y)=>x.speed-y.speed).slice(0,2); let r=[]; at.forEach(att=>{df.forEach(def=>{if(parseFloat(att.speed)-parseFloat(def.speed)>5) r.push({m:`${att.customName} (${att.speed}) ⚡ ${def.customName} (${def.speed})`,a:`Sub ${def.customName}`})})}); return r;}; const all=[...logic(teamB,teamA),...logic(teamA,teamB)]; if(all.length===0) c.innerHTML=`<div class="p-3 bg-green-900/20 text-center text-sm text-green-300">Teams balanced.</div>`; else all.slice(0,3).forEach(s=>c.innerHTML+=`<div class="bg-red-900/20 border-l-4 border-red-500 p-3 rounded-r flex gap-3 text-sm"><span class="text-xl">⚠️</span><div><p class="font-bold text-red-200">${s.m}</p><p class="text-gray-400">REC: ${s.a}</p></div></div>`); }
        function renderPerf(){ 
            const s=(l)=>l.reduce((a,b)=>a+parseFloat(b.score),0); const sa=s(teamA), sb=s(teamB), t=sa+sb; if(t===0)return; const pa=Math.round((sa/t)*100), pb=100-pa;
            const bA=document.getElementById('barTeamA'), bB=document.getElementById('barTeamB');
            bA.style.width=`${pa}%`; bA.style.backgroundColor=teamColors.A.hex; bB.style.width=`${pb}%`; bB.style.backgroundColor=teamColors.B.hex;
            document.getElementById('pctTeamA').innerText=`${pa}%`; document.getElementById('pctTeamB').innerText=`${pb}%`;
            const d=Math.abs(pa-pb), txt=document.getElementById('verdictText');
            if(d<3){txt.innerText="Balanced Match";txt.style.color="#ccc";}else{const w=pa>pb?'A':'B';txt.style.color=w==='A'?teamColors.A.hex:teamColors.B.hex;txt.innerText=w==='A'?`Team A Dominating`:`Team B Dominating`;}
        }
        function toggleLanguage(){
            currentLang=currentLang==='en'?'ar':'en';
            document.documentElement.lang=currentLang;
            document.documentElement.dir=currentLang==='ar'?'rtl':'ltr';
            document.querySelectorAll('[data-i18n]').forEach(e=>e.innerText=dictionary[currentLang][e.getAttribute('data-i18n')]);
            updDrops();
            renderAttackScenarios();
            renderStrategicComments(); // Refresh comments on language toggle
        }
        function updDrops(){const s=document.getElementById('teamSelector'),v=s.value,nA=document.getElementById('lblTeamADash').innerText,nB=document.getElementById('lblTeamBDash').innerText;s.innerHTML=`<option value="A">${dictionary[currentLang].selectOptA} (${nA})</option><option value="B">${dictionary[currentLang].selectOptB} (${nB})</option>`;s.value=v||'A';const o=[{v:'rating',t:dictionary[currentLang].colRating},{v:'skill',t:dictionary[currentLang].colSkill},{v:'speed',t:dictionary[currentLang].colSpeed}];['A','B'].forEach(t=>{const e=document.getElementById(`sort${t}`);if(e){const c=e.value||'rating';e.innerHTML=o.map(x=>`<option value="${x.v}">${x.t}</option>`).join('');e.value=c;}});}
        function handleSortChange(t){currentSort[t]=document.getElementById(`sort${t}`).value;renderT();}
        
        currentLang='en'; 
    </script>
</body>
</html>
