<script>
	import { onMount } from 'svelte';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Input } from '$lib/components/ui/input/index.js';

	// List of prime numbers between 0 and 509
	const primeNumbers = [
		28, 31, 36, 123, 130, 166, 210, 230, 262, 270, 316, 319, 333, 346, 356, 379, 405, 444, 451, 459,
		485, 491, 554, 580, 592, 593, 617, 626, 646, 656, 662, 679, 710, 730, 741, 803, 853, 879, 883,
		889, 903, 930, 936, 942, 950, 964, 978, 980, 1004, 1021
	];

	let userInput = ''; // Input from the user
	let revealedPrizes = []; // Array to track revealed numbers for reactivity
	let feedback = ''; // Feedback message for user
	let feedbackColor = ''; // Feedback color based on result

	// States for hints
	let showHint1 = false;
	let showHint2 = false;
	let showHint3 = false;

	// Reference to the input field for focusing
	let inputRef;

	// Function to handle user input submission
	function checkPrize(event) {
		if (event) {
			event.preventDefault(); // Prevent form submission
		}
		const number = parseInt(userInput);
		if (primeNumbers.includes(number)) {
			if (!revealedPrizes.includes(number)) {
				revealedPrizes = [...revealedPrizes, number]; // Update the array to trigger reactivity
				feedback = `El numero ${number} es ganador!`;
				feedbackColor = 'text-green-500';
			} else {
				feedback = `El numero ${number} ya fue revelado.`;
				feedbackColor = 'text-yellow-500';
			}
		} else {
			feedback = `El numero ${number} no es ganador.`;
			feedbackColor = 'text-red-500';
		}
		userInput = ''; // Reset the input field
		inputRef.focus(); // Refocus the input field
	}

	// Generate a list of numbers from 0 to the max prime number
	let allNumbers = [];
	onMount(() => {
		allNumbers = primeNumbers; // Only show prime numbers in the grid
	});

	// Functions to toggle hints
	function toggleHint1() {
		showHint1 = !showHint1;
	}

	function toggleHint2() {
		showHint2 = !showHint2;
	}

	function toggleHint3() {
		showHint3 = !showHint3;
	}
</script>

<!-- Main Container -->
<div class="flex flex-col items-center space-y-4 p-4 text-2xl">
	<!-- Input bar for user to submit numbers -->
	<form class="flex w-full max-w-sm items-center space-x-2" on:submit|preventDefault={checkPrize}>
		<Input type="number" bind:value={userInput} placeholder="Enter number" bind:this={inputRef} />
		<Button type="submit">Submit</Button>
	</form>

	<!-- Feedback message -->
	<p class="{feedbackColor} text-lg font-semibold">{feedback}</p>

	<!-- Flex container of boxes that wraps -->
	<div class="flex w-3/4 flex-wrap gap-2">
		{#each allNumbers as number}
			<div
				class="flex h-20 w-20 items-center justify-center rounded font-bold text-white
					{revealedPrizes.includes(number) ? 'bg-green-500' : 'bg-gray-700'}"
			>
				{revealedPrizes.includes(number) ? number : ''}
			</div>
		{/each}
	</div>

	<!-- Hint Section -->
	<div class="mt-4 flex items-start space-x-10">
		<div class="flex flex-col gap-4">
			<Button on:click={toggleHint1}>Pista 1</Button>
			{#if showHint1}
				<p class="text-lg">314</p>
			{/if}
		</div>
		<div class="flex flex-col gap-4">
			<Button on:click={toggleHint2}>Pista 2</Button>
			{#if showHint2}
				<p class="text-lg">En Binario, 10 bits</p>
			{/if}
		</div>
		<div class="flex flex-col gap-4">
			<Button on:click={toggleHint3}>Pista 3</Button>
			{#if showHint3}
				<p class="text-lg">0100111010</p>
			{/if}
		</div>
	</div>
</div>
