<script>
  // Import the Svelte store module
  import { writable } from 'svelte/store';

  // Create a store to hold the pomodoro time in seconds
  const pomodoroTime = writable(25 * 60);

  // Create a store to hold the break time in seconds
  const breakTime = writable(5 * 60); 

  // Create a store to hold the number of completed pomodoros
  const pomodoroCount = writable(0);

  // Create a store to hold the timer state
  // Possible values are 'idle', 'active', 'break'
  const timerState = writable('idle');

  // Create a store to hold the timer interval
  const timerInterval = writable(null);

  // Create a function to start the timer
  function startTimer() {
	// Set the timer state to 'active'
	timerState.set('active');

	// Set the timer interval to decrement the pomodoro time every second
	timerInterval.set(
	  setInterval(() => {
		pomodoroTime.update((time) => {
		  // If the time reaches zero, stop the timer and start the break
		  if (time === 0) {
			stopTimer();
			startBreak();
			return time;
		  }

		  // Otherwise, return the decremented time
		  return time - 1;
		});
	  }, 1000)
	);
  }

  // Create a function to stop the timer
  function stopTimer() {
	// Set the timer state to 'idle'
	timerState.set('idle');

	// Clear the timer interval
	clearInterval($timerInterval);

	// Reset the timer interval to null
	timerInterval.set(null);
  }

  // Create a function to reset the timer
  function resetTimer() {
	// Stop the timer
	stopTimer();

	// Reset the pomodoro time to 25 minutes
	pomodoroTime.set(25 * 60);

	// Reset the break time to 5 minutes
	breakTime.set(5 * 60);

	// Reset the pomodoro count to zero
	pomodoroCount.set(0);
  }

  // Create a function to start the break
  function startBreak() {
	// Set the timer state to 'break'
	timerState.set('break');

	// Increment the pomodoro count
	pomodoroCount.update((count) => count + 1);

	// Set the break time to 20 minutes if the pomodoro count is a multiple of 4
	// Otherwise, set it to 5 minutes
	if ($pomodoroCount % 4 === 0) {
	  breakTime.set(20 * 60);
	} else {
	  breakTime.set(5 * 60);
	}

	// Set the timer interval to decrement the break time every second
	timerInterval.set(
	  setInterval(() => {
		breakTime.update((time) => {
		  // If the time reaches zero, stop the timer and start a new pomodoro
		  if (time === 0) {
			stopTimer();
			startTimer();
			return time;
		  }

		  // Otherwise, return the decremented time
		  return time - 1;
		});
	  }, 1000)
	);
  }

  // Create a function to format the time in minutes and seconds
  function formatTime(timeInSeconds) {
	const minutes = Math.floor(timeInSeconds / 60);
	const seconds = timeInSeconds % 60;
	return `${minutes.toString().padStart(2, '0')}:${seconds
	  .toString()
	  .padStart(2, '0')}`;
  }
</script>

<style>

	
	.inter-modoro {
	  font-family: "Inter", sans-serif;
	  font-optical-sizing: auto;
	  font-weight: 800; 
	  font-style: normal;
	  font-variation-settings:
		"slnt" 0;
	}
	
  .timer {
	
	font-size: 3rem;
	text-align: center;
  }

  .buttons {
	display: flex;
	justify-content: center;
	gap: 1rem;
  }

  button {
	padding: 0.5rem 1rem;
	border: none;
	border-radius: 0.5rem;
	background-color: #f0f0f0;
	cursor: pointer;
	font-family: "Inter", sans-serif;
	transition: transform .2s
  }

  button:hover {
	background-color: #e0e0e0;
	transform: scale(1.2)
  }

  button:active {
	background-color: #d0d0d0;
	transform: scale(1.1)
  }

  button:focus {
	outline: none;
	box-shadow: 0 0 0 2px #280;
  }

  .idle {
	color: #fff;
  }
  
  .count{
	color: #fff;
  }

  .active {
	color: #6a6;
  }

  .break {
	color: #b55;
  }
</style>

<div class="timer inter-modoro">
  <!-- Show the pomodoro time or the break time depending on the timer state -->
  {#if $timerState === 'active'}
	<p class="active">{formatTime($pomodoroTime)}</p>
  {:else if $timerState === 'break'}
	<p class="break">{formatTime($breakTime)}</p>
  {:else}
	<p class="idle">{formatTime($pomodoroTime)}</p>
  {/if}

  <!-- Show the pomodoro count -->
  <p class="count">Pomodoros: {$pomodoroCount}</p>
</div>

<div class="buttons">
  <!-- Show the start, stop, and reset buttons -->
  <button on:click={startTimer}>Start</button>
  <button on:click={stopTimer}>Stop</button>
  <button on:click={resetTimer}>Reset</button>
</div>
