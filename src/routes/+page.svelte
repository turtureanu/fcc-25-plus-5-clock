<script lang="ts">
	let breakLength = $state(5); // in minutes
	let sessionLength = $state(25); // in minutes
	let isStopped = $state(true);
	// svelte-ignore state_referenced_locally
	let timeLeft: number = $state(sessionLength * 60); // in seconds
	let timerLabel = $state('Session');
	let isBreak = false;
	let countdown: number | undefined;

	const count = () => {
		isStopped = !isStopped;
		if (!isStopped) {
			if (timeLeft === 0) {
				document.getElementById('beep') && document.getElementById('beep').play();
				// wait a second at 00:00 before switching (break/session)
				setTimeout(() => {
					if (isBreak) {
						timeLeft = breakLength * 60;
					} else {
						timeLeft = sessionLength * 60;
					}

					setCountdown();
				}, 1000);
			} else {
				setCountdown();
			}
		} else {
			clearInterval(countdown);
		}
	};

	const setCountdown = () => {
		countdown = setInterval(() => {
			timeLeft > 0 && timeLeft--;

			if (timeLeft === 0) {
				isBreak = !isBreak;
				if (isBreak) {
					timerLabel = 'Break';
				} else {
					timerLabel = 'Session';
				}
				clearInterval(countdown);
				isStopped = !isStopped;
				count();
			}
		}, 1000);
	};

	// svelte-ignore state_referenced_locally
	let selectedTime: number = $state(sessionLength);

	$effect(() => {
		selectedTime = isBreak ? breakLength : sessionLength;
	});

	$effect(() => {
		timeLeft = selectedTime * 60;
	});

	let minutes = $state(() => {
		let min = 0;
		if (timeLeft % 60 === 0) {
			return timeLeft / 60 < 10 ? '0' + timeLeft / 60 : timeLeft / 60;
		}

		min = Math.floor(timeLeft / 60);

		if (min < 10) {
			return '0' + min;
		}

		return min;
	});

	let seconds = $state(() => {
		let sec = timeLeft % 60;
		if (sec < 10) {
			return '0' + (timeLeft % 60);
		}

		return sec;
	});
</script>

<div class="m-auto flex h-full max-w-md flex-col justify-center p-8">
	<audio
		id="beep"
		class="hidden"
		src="https://cdn.freecodecamp.org/testable-projects-fcc/audio/BeepSound.wav"
	></audio>
	<div class="flex flex-col items-center justify-center">
		<div class="text-8xl" id="time-left">
			{minutes()}:{seconds()}
		</div>
	</div>

	<p class="mb-24 mt-4 text-center text-xl uppercase" id="timer-label">{timerLabel}</p>
	<div class="mb-8 flex max-w-md justify-evenly">
		<div class="flex w-fit flex-col items-center">
			<p class="text-md mb-2 uppercase" id="break-label">Break Length</p>
			<div class="flex w-fit items-center rounded-md bg-zinc-900 p-2">
				<button
					onclick={() => isStopped && breakLength > 1 && breakLength--}
					class="h-10 w-10 cursor-pointer text-xl"
					id="break-decrement">-</button
				>
				<div class="mx-2 text-2xl" id="break-length">{breakLength}</div>
				<button
					onclick={() => isStopped && breakLength < 60 && breakLength++}
					class="h-10 w-10 cursor-pointer text-xl"
					id="break-increment">+</button
				>
			</div>
		</div>
		<div class="flex w-fit flex-col items-center">
			<p class="text-md mb-2 uppercase" id="session-label">Session Length</p>
			<div class="flex w-fit items-center rounded-md bg-zinc-900 p-2">
				<button
					onclick={() => isStopped && sessionLength > 1 && sessionLength--}
					class="h-10 w-10 cursor-pointer text-xl"
					id="session-decrement">-</button
				>
				<div class="mx-2 text-2xl" id="session-length">{sessionLength}</div>
				<button
					onclick={() => isStopped && sessionLength < 60 && sessionLength++}
					class="h-10 w-10 cursor-pointer text-xl"
					id="session-increment">+</button
				>
			</div>
		</div>
	</div>
	<div class="mx-10 flex">
		<button
			onclick={() => count()}
			class="h-full w-[50%] cursor-pointer py-4 text-xl uppercase"
			id="start_stop">{isStopped ? 'Start' : 'Stop'}</button
		>
		<button
			onclick={() => {
				clearInterval(countdown);
				document.getElementById('beep') && document.getElementById('beep').pause();
				document.getElementById('beep') && (document.getElementById('beep').currentTime = 0);
				isStopped = true;
				isBreak = false;
				breakLength = 5;
				sessionLength = 25;
				timeLeft = sessionLength * 60;
				timerLabel = 'Session';
			}}
			class="h-full w-[50%] cursor-pointer py-4 text-xl uppercase"
			id="reset">Reset</button
		>
	</div>
</div>
