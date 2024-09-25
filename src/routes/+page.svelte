<script>
	import { onMount } from 'svelte';

	// Inicializar las variables del juego
	let currentQuestionIndex = 0; // Rastrea la pregunta actual
	let currentQuestion = '';
	let currentValue = 50000000; // Valor inicial
	let minValue = 0;
	let maxValue = 100000000;
	let threshold = 100000; // Umbral de precisión para la búsqueda binaria
	let currentPath = '';
	let branches = []; // Almacena las ramas dibujadas
	let finalMessage = '';
	let questionResults = []; // Almacena el resultado de cada pregunta

	// Array de preguntas
	const questions = [
		{ text: 'Estarías dispuesto a perder un brazo por', unit: ' pesos?' },
		{ text: 'Volverías a empezar tu carrera por', unit: ' pesos?' },
		{ text: 'Te tatuarías el nombre de tu ex en la frente por', unit: ' pesos?' },
		{
			text: 'Te irias a vivir a una isla desierta sin tecnologia ni personas por 5 años por',
			unit: ' pesos?'
		}
	];

	// Ángulos aleatorios para las ramas izquierda y derecha
	let leftAngleDegrees = 30;
	let rightAngleDegrees = 30;
	let leftAngle = leftAngleDegrees * (Math.PI / 180);
	let rightAngle = rightAngleDegrees * (Math.PI / 180);

	// Crear una estructura inicial de ramas
	onMount(() => {
		generateTree();
		askNextQuestion();
	});

	// Dibujar una rama basada en el camino del jugador
	function drawBranch(x1, y1, angle, length, depth, path) {
		const x2 = x1 + length * Math.cos(angle);
		const y2 = y1 + length * Math.sin(angle);
		branches.push({ x1, y1, x2, y2, depth, path, questionIndex: currentQuestionIndex });
		return { x2, y2 };
	}

	// Generar el árbol basado en el camino actual
	function generateTree() {
		let x1 = 0,
			y1 = 40;
		let length = 20;
		let angle = -Math.PI / 2;
		let depth = 1;
		let path = '';
		let newCoords = drawBranch(x1, y1, angle, length, depth, path);

		for (const direction of currentPath) {
			length *= 0.8;
			depth += 1;
			angle += direction === 'L' ? -leftAngle : rightAngle;
			newCoords = drawBranch(newCoords.x2, newCoords.y2, angle, length, depth, path + direction);
			path += direction;
		}
	}

	// Hacer la siguiente pregunta basada en el rango actual de valores
	function askNextQuestion() {
		if (currentQuestionIndex < questions.length) {
			const question = questions[currentQuestionIndex];
			currentQuestion = `${question.text} ${currentValue.toLocaleString()}${question.unit}`;
		}
	}

	// Manejar la respuesta de búsqueda binaria
	function nextQuestion(choice) {
		if (choice === 'L') {
			// Si el jugador está dispuesto, reducir el valor máximo
			maxValue = currentValue;
			currentPath += 'L';
		} else {
			// Si no está dispuesto, aumentar el valor mínimo
			minValue = currentValue;
			currentPath += 'R';
		}

		// Reducir el valor y hacer la siguiente pregunta
		currentValue = Math.floor((maxValue + minValue) / 2);

		// Comprobar si la diferencia entre max y min es lo suficientemente pequeña para detenerse
		if (Math.abs(maxValue - minValue) <= threshold) {
			finalMessage = `Aceptarías por aproximadamente ${currentValue.toLocaleString()} pesos`;
			questionResults.push({ question: questions[currentQuestionIndex].text, value: currentValue });
		} else {
			askNextQuestion();
		}

		generateTree();
	}

	// Proceder a la siguiente pregunta
	function continueToNextQuestion() {
		currentQuestionIndex++;
		resetValues();
		if (currentQuestionIndex < questions.length) {
			askNextQuestion();
			generateTree();
		}
	}

	// Restablecer los valores para la siguiente pregunta
	function resetValues() {
		minValue = 0;
		maxValue = 100000000;
		currentValue = Math.floor((maxValue + minValue) / 2);
		currentPath = '';
		finalMessage = '';
	}

	// Restablecer todo el juego
	function resetGame() {
		currentQuestionIndex = 0;
		questionResults = [];
		resetValues();
		branches = [];
		generateTree();
		askNextQuestion();
	}
	generateTree();
</script>

<!-- Contenedor Principal -->
<div
	class="flex h-screen flex-col items-center justify-center bg-gradient-to-b from-indigo-500 to-purple-700 p-6 font-mono text-white"
>
	<h1 class="mb-8 text-4xl font-bold">Fractalizados!</h1>

	<!-- Mostrar la visualización del árbol -->
	<div class="relative mb-8 w-full max-w-screen-md grow overflow-hidden">
		<svg viewBox="-50 -50 100 100" class="absolute inset-0 h-full w-full">
			{#each branches as { x1, y1, x2, y2, depth, path, questionIndex }}
				<line
					{x1}
					{y1}
					{x2}
					{y2}
					stroke={questionIndex != currentQuestionIndex
						? 'black'
						: path === currentPath
							? 'white'
							: 'gray'}
					stroke-linecap="round"
					stroke-width={2 / depth}
				/>
			{/each}
		</svg>
	</div>

	<!-- Mostrar la pregunta actual o mensaje final -->
	{#if finalMessage === '' && currentQuestionIndex < questions.length}
		<div class="mb-4 text-center text-xl">
			<p>{currentQuestion}</p>
		</div>

		<!-- Botones para sí/no (decisión binaria) -->
		<div class="flex gap-4">
			<button
				on:click={() => nextQuestion('L')}
				class="rounded-lg bg-green-500 px-4 py-2 hover:bg-green-600"
			>
				Sí
			</button>
			<button
				on:click={() => nextQuestion('R')}
				class="rounded-lg bg-red-500 px-4 py-2 hover:bg-red-600"
			>
				No
			</button>
		</div>
	{:else if finalMessage !== '' && currentQuestionIndex < questions.length}
		<div class="text-center text-2xl">
			<p>{finalMessage}</p>
			<button
				on:click={continueToNextQuestion}
				class="mt-4 rounded-lg bg-blue-500 px-4 py-2 hover:bg-blue-600"
			>
				Continuar a la siguiente pregunta
			</button>
		</div>
	{:else}
		<div class="text-center text-2xl">
			<p class="mb-4">¡Juego Terminado! Aquí están tus resultados:</p>
			<ul class="text-left">
				{#each questionResults as { question, value }}
					<li class="mb-2">
						<strong>{question}</strong>: {value.toLocaleString()} pesos
					</li>
				{/each}
			</ul>
			<button on:click={resetGame} class="mt-4 rounded-lg bg-blue-500 px-4 py-2 hover:bg-blue-600">
				Empezar de nuevo
			</button>
		</div>
	{/if}
</div>
