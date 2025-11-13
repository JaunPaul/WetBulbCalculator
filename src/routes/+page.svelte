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
		content="Wet bulb calculator tuned for tobacco curing barns using live temperature and humidity sensor readings. Designed by Edge Works."
	/>
	<meta
		name="keywords"
		content="wet bulb, tobacco curing, barn, temperature, humidity, sensor, calculator, edge works"
	/>
	<meta
		name="author"
		content="Edge Works"
	/>
</svelte:head>

<div class="bg-gradient-to-b from-rock-spray-50 via-white to-white text-stone-900 dark:from-stone-950 dark:via-stone-900 dark:to-stone-950 dark:text-stone-100">
	<div class="mx-auto flex min-h-svh max-w-4xl flex-col px-4 pb-10 pt-14 sm:px-6 lg:px-8">
		<header class="flex items-start justify-between gap-4">
			<div>
				<p class="text-xs uppercase tracking-[0.3em] text-rock-spray-500 dark:text-rock-spray-400">Curing Monitor</p>
				<h1 class="mt-2 text-3xl font-semibold leading-tight tracking-tight sm:text-4xl">
					Tobacco Barn Wet Bulb Calculator
				</h1>
			</div>


		</header>

		<main class="mt-10 flex flex-1 flex-col gap-10 md:mt-14 md:flex-row">
			<section class="flex-1 space-y-6 rounded-3xl border border-stone-200/60 bg-white/90 p-6 shadow-xl shadow-stone-200/60 backdrop-blur transition dark:border-stone-800 dark:bg-stone-700/60 dark:shadow-none sm:p-8">
				<div>
					<h2 class="text-sm font-semibold uppercase tracking-[0.3em] text-rock-spray-700 dark:text-stone-100">
						Sensor Inputs
					</h2>
					<p class="mt-1 text-sm text-rock-spray-950/90 dark:text-stone-100">
						Set the current dry bulb and relative humidity readings from the curing barn sensors to see the live wet bulb value.
					</p>
				</div>

				<div class="space-y-5">
					<label class="block space-y-2">
						<div class="flex items-center justify-between text-xs font-medium uppercase tracking-[0.2em] text-rock-spray-950 dark:text-stone-100">
							<span>Dry Bulb Temperature</span>
							<span class="font-normal text-stone-100">°C</span>
						</div>
						<input
							type="number"
							class="w-full rounded-2xl border border-stone-200 bg-white px-4 py-3 text-base font-medium text-stone-900 shadow-sm outline-hidden transition placeholder:text-stone-100 focus:border-rock-spray-400 focus:shadow-md focus:ring-2 focus:ring-rock-spray-200 dark:border-stone-700 dark:bg-stone-900 dark:text-stone-100 dark:focus:border-rock-spray-500 dark:focus:ring-rock-spray-500/30"
							bind:value={temperature}
							step="0.1"
							min={TEMPERATURE_RANGE.min}
							max={TEMPERATURE_RANGE.max}
						/>
						<input
							type="range"
							class="w-full accent-rock-spray-500"
							bind:value={temperature}
							min={TEMPERATURE_RANGE.min}
							max={TEMPERATURE_RANGE.max}
							step="0.1"
						/>
						<p class="text-xs text-rock-spray-800 dark:text-stone-500">
							Common curing ramp range: 18° to 38°C
						</p>
					</label>

					<label class="block space-y-2">
						<div class="flex items-center justify-between text-xs font-medium uppercase tracking-[0.2em] text-rock-spray-950 dark:text-stone-100">
							<span>Relative Humidity</span>
							<span class="font-normal text-stone-100">%</span>
						</div>
						<input
							type="number"
							class="w-full rounded-2xl border border-stone-200 bg-white px-4 py-3 text-base font-medium text-stone-900 shadow-sm outline-hidden transition placeholder:text-stone-100 focus:border-rock-spray-400 focus:shadow-md focus:ring-2 focus:ring-rock-spray-200 dark:border-stone-700 dark:bg-stone-900 dark:text-stone-100 dark:focus:border-rock-spray-500 dark:focus:ring-rock-spray-500/30"
							bind:value={humidity}
							step="1"
							min={HUMIDITY_RANGE.min}
							max={HUMIDITY_RANGE.max}
						/>
						<input
							type="range"
							class="w-full accent-rock-spray-400 focus:ring-rock-spray-200 focus:ring-offset-2 dark:focus:ring-rock-spray-400/30"
							bind:value={humidity}
							min={HUMIDITY_RANGE.min}
							max={HUMIDITY_RANGE.max}
							step="1"
						/>
						<p class="text-xs text-rock-spray-800 dark:text-stone-500">Typical curing target: 55% to 75%</p>
					</label>
				</div>
			</section>

			<section class="flex-1 space-y-6 rounded-3xl border border-stone-200/60 bg-white/90 p-6 shadow-xl shadow-stone-200/60 backdrop-blur transition dark:border-stone-800 dark:bg-stone-900/60 dark:shadow-none sm:p-8">
				<div class="flex items-center justify-between gap-3">
					<h2 class="text-sm font-semibold uppercase tracking-[0.3em] text-rock-spray-700 dark:text-stone-500">
						Result
					</h2>
					<span class="rounded-full border border-stone-200/60 bg-white px-3 py-1 text-xs font-semibold text-stone-500 shadow-sm dark:border-stone-700 dark:bg-stone-800/60 dark:text-stone-300">
						Realtime update
					</span>
				</div>

				<div class="space-y-6 rounded-3xl bg-gradient-to-br from-rock-spray-500 via-rock-spray-400 to-rock-spray-500 p-[1px]">
					<div class="flex flex-col gap-6 rounded-[calc(theme(spacing.6)*1.2)]  px-6 py-8 text-rock-spray-950 shadow-xl shadow-rock-spray-500/20 bg-rock-spray-200 dark:shadow-none">
						<div>
							<p class="text-sm uppercase tracking-[0.4em] text-rock-spray-950/90">Wet Bulb Target</p>
							<p class="mt-3 text-5xl font-bold tracking-tight sm:text-6xl">
								{formatNumber(wetBulb)}<span class="ml-2 text-lg font-medium text-rock-spray-950/90">°C</span>
							</p>
						</div>

						<div class="grid gap-4 text-sm text-stone-400 dark:text-stone-300 sm:grid-cols-2">
							<div class="rounded-2xl border border-stone-100/80 bg-white px-4 py-3 dark:border-stone-700/60 dark:bg-stone-800/40">
								<p class="text-xs uppercase tracking-[0.3em] text-rock-spray-950/90 dark:text-stone-500">
									Inputs
								</p>
								<p class="mt-2 font-medium text-rock-spray-600 dark:text-stone-300">
									{formatNumber(temperature)} °C
								</p>
								<p class="font-medium text-rock-spray-600 dark:text-stone-300">
									{formatNumber(humidity)} %
								</p>
							</div>

						</div>
					</div>
				</div>

				<div class="space-y-3 text-sm text-rock-spray-950/90 dark:text-stone-300">

					<p class="text-xs text-rock-spray-950/90 dark:text-stone-500">
						Formula based on the Stull approximation (2011). Accuracy ±1°C for 0°C ≤ T ≤ 50°C and 5% ≤ RH ≤ 99%.
					</p>
				</div>
			</section>
		</main>

		<footer class="mt-12 text-center text-xs text-rock-spray-950/90 dark:text-stone-500">
			<span>Designed by Edge Works · <a href="https://theedge.works" target="_blank">theedge.works</a> · Built with SvelteKit & Tailwind CSS</span>
		</footer>
	</div>
</div>
