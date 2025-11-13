<script lang="ts">
	import { browser } from '$app/environment';
	import { onDestroy, onMount } from 'svelte';

	let temperature = 32;
	let humidity = 60;
	let isDark = false;

	const TEMPERATURE_RANGE = { min: -40, max: 60 };
	const HUMIDITY_RANGE = { min: 0, max: 100 };

	let mediaQuery: MediaQueryList | null = null;
	let mediaQueryHandler: ((event: MediaQueryListEvent) => void) | null = null;

	onMount(() => {
		if (!browser) return;

		mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');

		const storedTheme = localStorage.getItem('theme');
		if (storedTheme === 'dark' || storedTheme === 'light') {
			isDark = storedTheme === 'dark';
		} else {
			isDark = mediaQuery.matches;
		}

		mediaQueryHandler = ({ matches }) => {
			if (localStorage.getItem('theme') === null) {
				isDark = matches;
			}
		};

		mediaQuery.addEventListener('change', mediaQueryHandler);
	});

	onDestroy(() => {
		if (mediaQuery && mediaQueryHandler) {
			mediaQuery.removeEventListener('change', mediaQueryHandler);
		}
	});

	function updateTheme() {
		if (!browser) return;

		const theme = isDark ? 'dark' : 'light';
		document.documentElement.classList.toggle('dark', isDark);
		document.documentElement.style.colorScheme = theme;
		localStorage.setItem('theme', theme);
	}

	$: updateTheme();

	function clearStoredTheme() {
		if (!browser) return;
		localStorage.removeItem('theme');
		isDark = mediaQuery?.matches ?? false;
	}

	const formatNumber = (value: number) =>
		Number.isFinite(value) ? value.toLocaleString(undefined, { maximumFractionDigits: 1 }) : '—';

	const clamp = (value: number, { min, max }: { min: number; max: number }) =>
		Math.min(Math.max(value, min), max);

	function calculateWetBulb(tempCelsius: number, relHumidity: number) {
		if (!Number.isFinite(tempCelsius) || !Number.isFinite(relHumidity)) return NaN;

		const T = clamp(tempCelsius, TEMPERATURE_RANGE);
		const RH = clamp(relHumidity, HUMIDITY_RANGE);

		const rhSqrt = Math.sqrt(RH + 8.313659);

		const wetBulb =
			T * Math.atan(0.151977 * rhSqrt) +
			Math.atan(T + RH) -
			Math.atan(RH - 1.676331) +
			0.00391838 * Math.pow(RH, 1.5) * Math.atan(0.023101 * RH) -
			4.686035;

		return wetBulb;
	}

	$: wetBulb = calculateWetBulb(temperature, humidity);
</script>

<svelte:head>
	<title>Wet Bulb Calculator</title>
	<meta
		name="description"
		content="Simple, modern wet bulb temperature calculator using air temperature and relative humidity."
	/>
</svelte:head>

<div class="bg-gradient-to-b from-sky-100 via-white to-white text-slate-900 dark:from-slate-950 dark:via-slate-900 dark:to-slate-950 dark:text-slate-100">
	<div class="mx-auto flex min-h-svh max-w-4xl flex-col px-4 pb-10 pt-14 sm:px-6 lg:px-8">
		<header class="flex items-start justify-between gap-4">
			<div>
				<p class="text-xs uppercase tracking-[0.3em] text-sky-500 dark:text-sky-400">Wet Bulb</p>
				<h1 class="mt-2 text-3xl font-semibold leading-tight tracking-tight sm:text-4xl">
					Ambient Comfort Calculator
				</h1>
			</div>

			<div class="flex items-center gap-2 rounded-full border border-slate-200/60 bg-white/80 p-1 shadow-sm shadow-slate-200 backdrop-blur dark:border-slate-700 dark:bg-slate-800/60 dark:shadow-none">
				<button
					type="button"
					class="rounded-full px-3 py-1 text-xs font-medium text-slate-500 transition hover:text-slate-900 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-sky-500 dark:text-slate-400 dark:hover:text-white"
					onclick={clearStoredTheme}
					aria-label="Reset to system theme"
				>
					System
				</button>
				<label class="flex cursor-pointer items-center gap-2 rounded-full bg-slate-100 px-2 py-1 text-xs font-medium text-slate-600 shadow-inner dark:bg-slate-700 dark:text-slate-200">
					<span>Dark</span>
					<input
						type="checkbox"
						class="size-4 accent-sky-500"
						bind:checked={isDark}
						aria-label="Toggle dark mode"
					/>
				</label>
			</div>
		</header>

		<main class="mt-10 flex flex-1 flex-col gap-10 md:mt-14 md:flex-row">
			<section class="flex-1 space-y-6 rounded-3xl border border-slate-200/60 bg-white/90 p-6 shadow-xl shadow-slate-200/60 backdrop-blur transition dark:border-slate-800 dark:bg-slate-900/60 dark:shadow-none sm:p-8">
				<div>
					<h2 class="text-sm font-semibold uppercase tracking-[0.3em] text-slate-400 dark:text-slate-500">
						Inputs
					</h2>
					<p class="mt-1 text-sm text-slate-500 dark:text-slate-400">
						Adjust the air temperature and relative humidity to instantly see the wet bulb temperature.
					</p>
				</div>

				<div class="space-y-5">
					<label class="block space-y-2">
						<div class="flex items-center justify-between text-xs font-medium uppercase tracking-[0.2em] text-slate-500 dark:text-slate-400">
							<span>Dry Bulb Temperature</span>
							<span class="font-normal text-slate-400">°C</span>
						</div>
						<input
							type="number"
							class="w-full rounded-2xl border border-slate-200 bg-white px-4 py-3 text-base font-medium text-slate-900 shadow-sm outline-hidden transition placeholder:text-slate-400 focus:border-sky-400 focus:shadow-md focus:ring-2 focus:ring-sky-200 dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100 dark:focus:border-sky-500 dark:focus:ring-sky-500/30"
							bind:value={temperature}
							step="0.1"
							min={TEMPERATURE_RANGE.min}
							max={TEMPERATURE_RANGE.max}
						/>
						<input
							type="range"
							class="w-full"
							bind:value={temperature}
							min={TEMPERATURE_RANGE.min}
							max={TEMPERATURE_RANGE.max}
							step="0.1"
						/>
						<p class="text-xs text-slate-400 dark:text-slate-500">
							Typical comfort range: 18° to 27°C
						</p>
					</label>

					<label class="block space-y-2">
						<div class="flex items-center justify-between text-xs font-medium uppercase tracking-[0.2em] text-slate-500 dark:text-slate-400">
							<span>Relative Humidity</span>
							<span class="font-normal text-slate-400">%</span>
						</div>
						<input
							type="number"
							class="w-full rounded-2xl border border-slate-200 bg-white px-4 py-3 text-base font-medium text-slate-900 shadow-sm outline-hidden transition placeholder:text-slate-400 focus:border-sky-400 focus:shadow-md focus:ring-2 focus:ring-sky-200 dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100 dark:focus:border-sky-500 dark:focus:ring-sky-500/30"
							bind:value={humidity}
							step="1"
							min={HUMIDITY_RANGE.min}
							max={HUMIDITY_RANGE.max}
						/>
						<input
							type="range"
							class="w-full"
							bind:value={humidity}
							min={HUMIDITY_RANGE.min}
							max={HUMIDITY_RANGE.max}
							step="1"
						/>
						<p class="text-xs text-slate-400 dark:text-slate-500">Comfort range: 30% to 60%</p>
					</label>
				</div>
			</section>

			<section class="flex-1 space-y-6 rounded-3xl border border-slate-200/60 bg-white/90 p-6 shadow-xl shadow-slate-200/60 backdrop-blur transition dark:border-slate-800 dark:bg-slate-900/60 dark:shadow-none sm:p-8">
				<div class="flex items-center justify-between gap-3">
					<h2 class="text-sm font-semibold uppercase tracking-[0.3em] text-slate-400 dark:text-slate-500">
						Result
					</h2>
					<span class="rounded-full border border-slate-200/60 bg-white px-3 py-1 text-xs font-semibold text-slate-500 shadow-sm dark:border-slate-700 dark:bg-slate-800/60 dark:text-slate-300">
						Realtime update
					</span>
				</div>

				<div class="space-y-6 rounded-3xl bg-gradient-to-br from-sky-500 via-sky-400 to-sky-500 p-[1px]">
					<div class="flex flex-col gap-6 rounded-[calc(theme(spacing.6)*1.2)] bg-white px-6 py-8 text-slate-900 shadow-xl shadow-sky-500/20 dark:bg-slate-950 dark:text-white dark:shadow-none">
						<div>
							<p class="text-sm uppercase tracking-[0.4em] text-sky-500 dark:text-sky-400">Wet Bulb</p>
							<p class="mt-3 text-5xl font-semibold tracking-tight sm:text-6xl">
								{formatNumber(wetBulb)}<span class="ml-2 text-lg font-medium text-sky-500 dark:text-sky-400">°C</span>
							</p>
						</div>

						<div class="grid gap-4 text-sm text-slate-600 dark:text-slate-300 sm:grid-cols-2">
							<div class="rounded-2xl border border-slate-100/80 bg-slate-50/80 px-4 py-3 dark:border-slate-700/60 dark:bg-slate-800/40">
								<p class="text-xs uppercase tracking-[0.3em] text-slate-400 dark:text-slate-500">
									Inputs
								</p>
								<p class="mt-2 font-medium">
									{formatNumber(temperature)} °C
								</p>
								<p class="font-medium text-slate-500 dark:text-slate-300">
									{formatNumber(humidity)} %
								</p>
							</div>
							<div class="rounded-2xl border border-slate-100/80 bg-slate-50/80 px-4 py-3 dark:border-slate-700/60 dark:bg-slate-800/40">
								<p class="text-xs uppercase tracking-[0.3em] text-slate-400 dark:text-slate-500">
									Feels Like
								</p>
								<p class="mt-2 font-medium">
									The wet bulb reflects how humid air limits cooling by evaporation.
								</p>
							</div>
						</div>
					</div>
				</div>

				<div class="space-y-3 text-sm text-slate-600 dark:text-slate-300">
					<p>
						The wet bulb temperature is the coolest temperature achievable through evaporative cooling.
						It is critical for assessing heat stress and HVAC efficiency.
					</p>
					<p class="text-xs text-slate-400 dark:text-slate-500">
						Formula based on the Stull approximation (2011). Accuracy ±1°C for 0°C ≤ T ≤ 50°C and 5% ≤ RH ≤ 99%.
					</p>
				</div>
			</section>
		</main>

		<footer class="mt-12 text-center text-xs text-slate-400 dark:text-slate-500">
			<span>Design by Wet Bulb Lab · Built with SvelteKit & Tailwind CSS</span>
		</footer>
	</div>
</div>
