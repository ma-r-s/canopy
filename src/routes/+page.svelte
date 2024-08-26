<script>
	import { Confetti } from 'svelte-confetti';
	import { Button } from '$lib/components/ui/button';

	// State variables
	let currentQuestionIndex = 0;
	let currentPath = '';
	let branches = []; // Store branches with start and end points
	let finalMessage = ''; // Message to display at the end of the game
	let prizeMessage = ''; // Message for winning a prize

	const questions = [
		{ question: 'Cual empanada es mejor?', leftAnswer: 'Saudade', rightAnswer: 'Típicas' },
		{ question: 'Tener clase en ...', leftAnswer: 'El SD', rightAnswer: 'El O' },
		{
			question: 'Hacer trabajos en...',
			leftAnswer: 'Biblioteca del ML',
			rightAnswer: 'Biblioteca del SD'
		},
		{ question: 'Para tu clase vas por:', leftAnswer: 'Ascensor', rightAnswer: 'Escaleras' },
		{ question: 'En el salón te haces:', leftAnswer: 'Adelante', rightAnswer: 'Atrás' },
		{ question: 'Llegas a la U en:', leftAnswer: 'Wheels', rightAnswer: 'Transmi' },
		{
			question: 'Estudias para el parcial:',
			leftAnswer: 'Una semana antes',
			rightAnswer: 'El día antes'
		},
		{ question: 'Tu almuerzo:', leftAnswer: 'Lo traes', rightAnswer: 'Lo compras' },
		{ question: 'Vas a clase:', leftAnswer: 'En la mañana', rightAnswer: 'En la tarde' }
	];

	let branchAngleDegrees = 20; // Default branch angle in degrees
	let branchAngle = branchAngleDegrees * (Math.PI / 180); // Angle in radians

	// List of prime numbers between 0 and 255
	const primeNumbers = [
		2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97,
		101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173, 179, 181, 191, 193,
		197, 199, 211, 223, 227, 229, 233, 239, 241, 251
	];

	// Function to generate a random color in hex format
	function getRandomColor() {
		const letters = '0123456789ABCDEF';
		let color = '#';
		for (let i = 0; i < 6; i++) {
			color += letters[Math.floor(Math.random() * 16)];
		}
		return color;
	}

	// Function to draw a branch
	function drawBranch(x1, y1, angle, length, depth, path) {
		const x2 = x1 + length * Math.cos(angle);
		const y2 = y1 + length * Math.sin(angle);

		branches.push({ x1, y1, x2, y2, depth, path });
		return { x2, y2 };
	}

	// Function to generate the tree based on the current path and angle
	function generateTree() {
		// Initial branch setup
		let x1 = 0,
			y1 = 40;
		let length = 20;
		let angle = -Math.PI / 2; // Start with an angle pointing up (-90 degrees)
		let depth = 1;

		let path = ''; // Start with an empty path
		let newCoords = drawBranch(x1, y1, angle, length, depth, path);

		for (const direction of currentPath) {
			length *= 0.8;
			depth += 1;
			angle += direction === 'L' ? -branchAngle : branchAngle;
			newCoords = drawBranch(newCoords.x2, newCoords.y2, angle, length, depth, path + direction);
			path += direction;
		}
	}

	// Function to calculate the binary number from the current path
	function calculateBinaryPath(path) {
		let binary = 0;
		for (let i = 0; i < path.length; i++) {
			if (path[i] === 'R') {
				binary += 1 << (path.length - 1 - i);
			}
		}
		return binary;
	}

	// Handle question answering and drawing branches
	function nextQuestion(choice) {
		const newPath = currentPath + choice;
		currentPath = newPath;
		generateTree(); // Redraw the tree after each choice

		if (currentQuestionIndex < questions.length - 1) {
			currentQuestionIndex++;
		} else {
			// End of questions
			const binaryValue = calculateBinaryPath(currentPath);
			finalMessage = `Camino: ${binaryValue}`;
			if (binaryValue === 42) {
				prizeMessage = 'gran';
			} else if (primeNumbers.includes(binaryValue)) {
				prizeMessage = 'peq';
			} else {
				prizeMessage = ''; // No prize if not a prime number or 42
			}
			currentQuestionIndex = -1;
		}
	}

	// Function to reset the game
	function resetGame() {
		currentQuestionIndex = 0;
		currentPath = '';
		finalMessage = ''; // Reset final message
		prizeMessage = ''; // Reset prize message
		generateTree();
	}

	// Generate the initial tree and random gradient
	let gradientStart = getRandomColor();
	let gradientEnd = getRandomColor();
	let gradientStyle = `background: linear-gradient(to bottom, ${gradientStart}, ${gradientEnd});`;

	generateTree();
</script>

<!-- Main container -->
<div
	class="h-dvh select-none flex-col items-center justify-center p-6 font-mono"
	style={gradientStyle}
>
	<!-- Game Title -->
	<h1 class="font-custom mb-8 text-4xl text-white">Búsqueda Fractal</h1>

	<!-- SVG to render the tree -->
	<div class="relative w-full max-w-screen-md grow overflow-hidden">
		<svg viewBox="-50 -50 100 100" class="absolute inset-0 h-full w-full">
			{#each branches as { x1, y1, x2, y2, depth, path }}
				<line
					{x1}
					{y1}
					{x2}
					{y2}
					stroke={path === currentPath ? 'black' : 'white'}
					stroke-linecap="round"
					stroke-width={2 / depth}
				/>
			{/each}
		</svg>
	</div>

	<!-- Display questions and buttons -->
	<div class="h-44">
		{#if currentQuestionIndex >= 0}
			<div class="mt-6 text-center text-2xl">
				<p class="mb-4 font-semibold text-white">
					{questions[currentQuestionIndex].question}
				</p>
				<div class="flex justify-center gap-4">
					<Button
						on:click={() => nextQuestion('L')}
						class="bg-emerald-500 text-white hover:bg-emerald-600"
					>
						{questions[currentQuestionIndex].leftAnswer}
					</Button>
					<Button
						on:click={() => nextQuestion('R')}
						class="bg-amber-500 text-white hover:bg-amber-600"
					>
						{questions[currentQuestionIndex].rightAnswer}
					</Button>
				</div>
			</div>
		{:else}
			<div class="mt-6 text-center">
				<p class="mb-4 text-3xl">{finalMessage}</p>
				{#if prizeMessage == 'peq'}
					<p class="mb-4 bg-yellow-300 p-4 font-bold text-black">🎉 Este numero tiene premio! 🎉</p>
				{:else if prizeMessage == 'gran'}
					<div class="flex">
						<Confetti infinite x={[-1, 1]} />

						<p class="mb-4 bg-yellow-300 p-1 text-3xl font-bold text-black">
							🎉 Encontraste el Gran Premio! 🎉
						</p>
						<Confetti infinite x={[-1, 1]} />
					</div>
				{:else}
					<p class="mb-4">Aquí no hay nada</p>
				{/if}
				<Button on:click={resetGame} class="">Empezar de nuevo</Button>
			</div>
		{/if}
	</div>
</div>
