🎵🧪 Musical Molecules

An interactive web application that makes learning basic chemistry fun and auditory! Build molecules atom by atom and hear unique synthesized sounds as chemical bonds are successfully formed, while learning about atomic valency.

✨ Features

Interactive Canvas: Build molecules by clicking and adding atoms to a dynamic, physics-based sandbox.

Auditory Feedback (Web Audio API): * Hear unique musical notes for different elements when a bond is successfully formed.

Hear distinct error tones if you try to break the rules of chemistry.

Valency Enforcement: The app strictly enforces chemical rules. For example, it will physically prevent you from adding more than 4 bonds to a Carbon atom or more than 2 bonds to an Oxygen atom.

Dynamic Formula Updates: The chemical formula updates in real-time as you construct your molecule (e.g., building water updates the formula to H₂O).

Physics Engine: Uses D3.js to simulate the forces between atoms, making the molecules look and act naturally connected.

🚀 How to Play

Start: Open the application. The canvas will be empty.

Drop an Atom: Click any element button (e.g., Oxygen O) on the left panel to drop the first atom onto the canvas. It will automatically be selected (indicated by a yellow ring).

Select an Atom: To add bonds to an existing atom, you must click it on the canvas to select it first.

Form Bonds: With an atom selected, click an element button on the left to bond a new atom to it.

Listen: Enjoy the chimes of successful bonds, but watch out for the error buzz if you exceed an atom's maximum valency!

🛠️ Technologies Used

This project is built entirely as a single-file, client-side web application.

HTML5 / CSS3 for structure and layout.

JavaScript (ES6+) for all logic and interactivity.

Tailwind CSS (via CDN) for rapid, responsive UI styling.

D3.js (Force-Directed Graph) for the physics engine and rendering the atoms/bonds.

Native Web Audio API for generating synthesized sound frequencies directly in the browser (no external audio files required!).

💻 Running the Project Locally

Because everything is contained within a single index.html file, running it is incredibly simple:

Clone or download this repository.

Open the index.html file directly in any modern web browser (Chrome, Firefox, Safari, Edge).

Note: Web Audio requires user interaction to start, so you won't hear sound until you click your first button.

🌐 Live Demo

You can try the live version of this project hosted on GitHub pages here:
[Insert Your GitHub Pages Link Here - e.g., https://yourusername.github.io/musical-molecules]

Created as a fun experiment combining web physics, generative audio, and basic chemistry.
