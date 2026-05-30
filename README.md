<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Musical Molecules</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- D3.js for force-directed graph/physics -->
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        .node circle { stroke: #fff; stroke-width: 2px; cursor: pointer; transition: stroke 0.2s, stroke-width 0.2s; }
        .node.selected circle { stroke: #fbbf24; stroke-width: 5px; } /* Amber-400 for selection */
        .node text { pointer-events: none; font-weight: bold; font-family: sans-serif; }
        .link { stroke: #9ca3af; stroke-width: 4px; stroke-linecap: round; }
        
        /* Custom animation for toast messages */
        @keyframes fadeOut {
            0% { opacity: 1; transform: translateY(0); }
            80% { opacity: 1; transform: translateY(0); }           100% { opacity: 0; transform: translateY(-10px); }
        }
        .toast { animation: fadeOut 2s forwards; }
    </style>
</head>
<body class="bg-gray-50 h-screen flex flex-col overflow-hidden">

    <!-- Header -->
    <header class="bg-indigo-600 text-white p-4 shadow-md z-10 flex justify-between items-center">
        <div>
            <h1 class="text-2xl font-bold">Musical Molecules 🎵🧪</h1>
            <p class="text-indigo-200 text-sm">Learn chemistry by hearing bonds form!</p>
        </div>
        <button id="resetBtn" class="bg-indigo-500 hover:bg-indigo-700 text-white px-4 py-2 rounded shadow transition">
            Reset Canvas
        </button>
    </header>

    <!-- Main Workspace -->
    <div class="flex-1 flex flex-col md:flex-row relative">
        <div class="w-full md:w-64 bg-white shadow-lg p-6 flex flex-col gap-4 z-10">
            <h2 class="font-semibold text-gray-700 mb-2 border-b pb-2">1. Select an atom on canvas</h2>
            <h2 class="font-semibold text-gray-700 mb-2">2. Add a new bond:</h2>
            
            <button onclick="addAtom('H')" class="flex items-center justify-between bg-gray-100 hover:bg-gray-200 text-gray-800 p-3 rounded-lg border border-gray-300 transition">
                <span class="flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-white border border-gray-400 flex items-center justify-center text-xs font-bold">H</span>
                    Hydrogen
                </span>
                <span class="text-xs text-gray-500">Max: 1</span>
            </button>
            
            <button onclick="addAtom('O')" class="flex items-center justify-between bg-red-50 hover:bg-red-100 text-red-800 p-3 rounded-lg border border-red-200 transition">
                <span class="flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-red-500 text-white flex items-center justify-center text-xs font-bold">O</span>
                    Oxygen
                </span>
                <span class="text-xs text-red-500">Max: 2</span>
            </button> <button onclick="addAtom('N')" class="flex items-center justify-between bg-blue-50 hover:bg-blue-100 text-blue-800 p-3 rounded-lg border border-blue-200 transition">
                <span class="flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-blue-500 text-white flex items-center justify-center text-xs font-bold">N</span>
                    Nitrogen
                </span>
                <span class="text-xs text-blue-500">Max: 3</span>
            </button>

            <button onclick="addAtom('C')" class="flex items-center justify-between bg-gray-700 hover:bg-gray-800 text-white p-3 rounded-lg border border-gray-900 transition mt-2">
                <span class="flex items-center gap-2">
                    <span class="w-6 h-6 rounded-full bg-gray-900 text-white flex items-center justify-center text-xs font-bold border border-gray-600">C</span>
                    Carbon
                </span>
                <span class="text-xs text-gray-300">Max: 4</span>
            </button>

            <div id="statsBox" class="mt-auto pt-4 border-t border-gray-200 text-sm text-gray-600">
                Formula: <span id="formula" class="font-bold text-indigo-600">C</span>
            </div>
        </div>

        <!-- Canvas Area -->
        <div id="canvasContainer" class="flex-1 bg-gray-50 relative cursor-crosshair"><!-- SVG injected by D3 -->
            
            <!-- Instructions Overlay -->
            <div id="instructions" class="absolute top-4 left-1/2 transform -translate-x-1/2 bg-white/80 backdrop-blur px-4 py-2 rounded-full shadow text-sm text-gray-600 pointer-events-none">
                Click the central Carbon atom to select it, then add elements!
            </div>

            <!-- Toast Container -->
            <div id="toastContainer" class="absolute bottom-10 left-1/2 transform -translate-x-1/2 flex flex-col gap-2 pointer-events-none z-50"></div>
        </div>
    </div>

    <script>
        // --- DATA & CONFIGURATION ---
        const elements = {
            'H': { valency: 1, color: '#ffffff', textColor: '#374151', radius: 15, mass: 1 },
            'O': { valency: 2, color: '#ef4444', textColor: '#ffffff', radius: 22, mass: 16 },
            'N': { valency: 3, color: '#3b82f6', textColor: '#ffffff', radius: 24, mass: 14 },
            'C': { valency: 4, color: '#1f2937', textColor: '#ffffff', radius: 26, mass: 12 }
        };

        let nodes = [];let links = [];
        let nodeIdCounter = 0;
        let selectedNodeId = null;

        // --- AUDIO SYSTEM (Web Audio API) ---
        let audioCtx = null;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        }

        function playSound(type, element = null) {
            initAudio();
            const osc = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            osc.connect(gainNode);
            gainNode.connect(audioCtx.destination);
const now = audioCtx.currentTime;

            if (type === 'success') {
                // Different elements have different notes (frequencies)
                let baseFreq = 440; // A4
                if (element === 'H') baseFreq = 880; // High pitch
                if (element === 'O') baseFreq = 523.25; // C5
                if (element === 'N') baseFreq = 659.25; // E5
                if (element === 'C') baseFreq = 329.63; // E4 (lower)

                osc.type = 'sine';
                osc.frequency.setValueAtTime(baseFreq, now);
                // Quick envelope for a "ping" sound
                gainNode.gain.setValueAtTime(0, now);
                gainNode.gain.linearRampToValueAtTime(0.3, now + 0.05);
                gainNode.gain.exponentialRampToValueAtTime(0.001, now + 0.5);
                
                osc.start(now);
                osc.stop(now + 0.5);
            } 
            else if (type === 'error') {
                osc.type = 'sawtooth';osc.frequency.setValueAtTime(150, now);
                osc.frequency.exponentialRampToValueAtTime(100, now + 0.3);
                
                // Buzzy error sound
                gainNode.gain.setValueAtTime(0.2, now);
                gainNode.gain.exponentialRampToValueAtTime(0.001, now + 0.3);
                
                osc.start(now);
                osc.stop(now + 0.3);
            }
            else if (type === 'select') {
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(600, now);
                gainNode.gain.setValueAtTime(0.05, now);
                gainNode.gain.exponentialRampToValueAtTime(0.001, now + 0.1);
                osc.start(now);
                osc.stop(now + 0.1);
            }
        }

        // --- UI UTILS ---
        function showToast(message, type = 'error') {
            const container = document.getElementById('toastContainer');const toast = document.createElement('div');
            
            const bgColor = type === 'error' ? 'bg-red-500' : 'bg-green-500';
            toast.className = `toast px-4 py-2 rounded shadow-lg text-white font-semibold text-sm ${bgColor}`;
            toast.textContent = message;
            
            container.appendChild(toast);
            setTimeout(() => toast.remove(), 2000);
        }

        function updateFormula() {
            const counts = { C: 0, H: 0, N: 0, O: 0 };
            nodes.forEach(n => counts[n.element]++);
            
            let formula = '';
            // Standard Hill system order: C, then H, then alphabetical
            ['C', 'H', 'N', 'O'].forEach(el => {
                if (counts[el] > 0) {
                    formula += el + (counts[el] > 1 ? `<sub>${counts[el]}</sub>` : '');
                }
            });
            document.getElementById('formula').innerHTML = formula || '-';
        }// --- GRAPH PHYSICS (D3.js) ---
        const container = document.getElementById('canvasContainer');
        const width = container.clientWidth;
        const height = container.clientHeight;

        const svg = d3.select("#canvasContainer")
            .append("svg")
            .attr("width", "100%")
            .attr("height", "100%");

        const simulation = d3.forceSimulation()
            .force("link", d3.forceLink().id(d => d.id).distance(60))
            .force("charge", d3.forceManyBody().strength(-400))
            .force("center", d3.forceCenter(width / 2, height / 2))
            .force("collide", d3.forceCollide().radius(d => elements[d.element].radius + 5));

        let linkSelection = svg.append("g").attr("class", "links").selectAll(".link");
        let nodeSelection = svg.append("g").attr("class", "nodes").selectAll(".node");

        function initializeGraph() { nodes = [{ id: nodeIdCounter++, element: 'C' }];
            links = [];
            selectedNodeId = nodes[0].id;
            updateGraph();
        }
