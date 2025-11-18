<script lang="ts">
	import { browser } from '$app/environment';
	import { onDestroy, onMount } from 'svelte';

	let temperature = 32;
	let humidity = 60;
	let isDark = false;

	const TEMPERATURE_RANGE = { min: -40, max: 100 };
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

<div class="bg-gradient-to-b from-torea-bay-50 via-white to-white text-slate-900 dark:from-slate-950 dark:via-slate-900 dark:to-slate-950 dark:text-slate-100">
	<div class="mx-auto flex min-h-svh max-w-4xl flex-col px-4 pb-10 pt-14 sm:px-6 lg:px-8">
		<div class="py-4 mb-4 flex items-center justify-between basis-1"><img src="/RLB_Group_Logo.svg" alt="RLB Group" class="h-16"></div>
		<header class="flex items-start justify-between gap-4">
			<div>
				<p class="text-xs uppercase tracking-[0.3em] text-torea-bay-500 dark:text-torea-bay-400">Curing Monitor</p>
				<h1 class="mt-2 text-3xl font-semibold leading-tight tracking-tight sm:text-4xl text-torea-bay-600">
					Tobacco Barn Wet Bulb Calculator
				</h1>
					<p class="mt-1 text-sm text-torea-bay-950/90 dark:text-slate-100">
					Brought to you by RLB Group. Precision Monitoring Made Simple.</p>
					<a href="tel:+263772320090" class="mt-4 inline-flex gap-2 px-2 py-1 rounded text-white bg-chateau-green-600 font-medium"><span><svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path fill="currentColor" d="M19.05 4.91A9.82 9.82 0 0 0 12.04 2c-5.46 0-9.91 4.45-9.91 9.91c0 1.75.46 3.45 1.32 4.95L2.05 22l5.25-1.38c1.45.79 3.08 1.21 4.74 1.21c5.46 0 9.91-4.45 9.91-9.91c0-2.65-1.03-5.14-2.9-7.01m-7.01 15.24c-1.48 0-2.93-.4-4.2-1.15l-.3-.18l-3.12.82l.83-3.04l-.2-.31a8.26 8.26 0 0 1-1.26-4.38c0-4.54 3.7-8.24 8.24-8.24c2.2 0 4.27.86 5.82 2.42a8.18 8.18 0 0 1 2.41 5.83c.02 4.54-3.68 8.23-8.22 8.23m4.52-6.16c-.25-.12-1.47-.72-1.69-.81c-.23-.08-.39-.12-.56.12c-.17.25-.64.81-.78.97c-.14.17-.29.19-.54.06c-.25-.12-1.05-.39-1.99-1.23c-.74-.66-1.23-1.47-1.38-1.72c-.14-.25-.02-.38.11-.51c.11-.11.25-.29.37-.43s.17-.25.25-.41c.08-.17.04-.31-.02-.43s-.56-1.34-.76-1.84c-.2-.48-.41-.42-.56-.43h-.48c-.17 0-.43.06-.66.31c-.22.25-.86.85-.86 2.07s.89 2.4 1.01 2.56c.12.17 1.75 2.67 4.23 3.74c.59.26 1.05.41 1.41.52c.59.19 1.13.16 1.56.1c.48-.07 1.47-.6 1.67-1.18c.21-.58.21-1.07.14-1.18s-.22-.16-.47-.28"/></svg></span>+263 772 320 090</a>
			</div>


		</header>

		<main class="mt-10 flex flex-1 flex-col gap-10 md:mt-14 md:flex-row">
			<section class="flex-1 space-y-6 rounded-3xl border border-torea-bay-400 bg-white/90 p-6 shadow-xl shadow-slate-200/60 backdrop-blur transition dark:border-slate-800 dark:bg-slate-700/60 dark:shadow-none sm:p-8">
				<div>
					<h2 class="text-sm font-semibold uppercase tracking-[0.3em] text-torea-bay-700 dark:text-slate-100">
						Sensor Inputs
					</h2>
					<p class="mt-1 text-sm text-torea-bay-950/90 dark:text-slate-100">
						Set the current temperature and relative humidity readings from the curing barn sensors to see the live wet bulb value.
					</p>
				</div>

				<div class="space-y-5">
					<label class="block space-y-2">
						<div class="flex items-center justify-between text-xs font-medium uppercase tracking-[0.2em] text-torea-bay-950 dark:text-slate-100">
							<span>Temperature</span>
							<span class="font-normal text-slate-100">°C</span>
						</div>
						<input
							type="number"
							class="w-full rounded-2xl border border-torea-bay-400 bg-white px-4 py-3 text-base font-medium text-slate-900 shadow-sm outline-hidden transition placeholder:text-slate-100 focus:border-torea-bay-400 focus:shadow-md focus:ring-2 focus:ring-torea-bay-200 dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100 dark:focus:border-torea-bay-500 dark:focus:ring-torea-bay-500/30"
							bind:value={temperature}
							step="0.1"
							min={TEMPERATURE_RANGE.min}
							max={TEMPERATURE_RANGE.max}
						/>
						<input
							type="range"
							class="w-full accent-chateau-green-500"
							bind:value={temperature}
							min={TEMPERATURE_RANGE.min}
							max={TEMPERATURE_RANGE.max}
							step="0.1"
						/>
						<p class="text-xs text-torea-bay-800 dark:text-slate-500">
							Common curing ramp range: 18° to 38°C
						</p>
					</label>

					<label class="block space-y-2">
						<div class="flex items-center justify-between text-xs font-medium uppercase tracking-[0.2em] text-torea-bay-950 dark:text-slate-100">
							<span>Relative Humidity</span>
							<span class="font-normal text-slate-100">%</span>
						</div>
						<input
							type="number"
							class="w-full rounded-2xl border border-torea-bay-400 bg-white px-4 py-3 text-base font-medium text-slate-900 shadow-sm outline-hidden transition placeholder:text-slate-100 focus:border-torea-bay-400 focus:shadow-md focus:ring-2 focus:ring-torea-bay-200 dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100 dark:focus:border-torea-bay-500 dark:focus:ring-torea-bay-500/30"
							bind:value={humidity}
							step="1"
							min={HUMIDITY_RANGE.min}
							max={HUMIDITY_RANGE.max}
						/>
						<input
							type="range"
							class="w-full accent-chateau-green-500 focus:ring-torea-bay-200 focus:ring-offset-2 dark:focus:ring-torea-bay-400/30"
							bind:value={humidity}
							min={HUMIDITY_RANGE.min}
							max={HUMIDITY_RANGE.max}
							step="1"
						/>
						<p class="text-xs text-torea-bay-800 dark:text-slate-500">Typical curing target: 55% to 75%</p>
					</label>
				</div>
			</section>

			<section class="flex-1 space-y-6 rounded-3xl border border-torea-bay-400 bg-white/90 p-6 shadow-xl shadow-slate-200/60 backdrop-blur transition dark:border-slate-800 dark:bg-slate-900/60 dark:shadow-none sm:p-8">
				<div class="flex items-center justify-between gap-3">
					<h2 class="text-sm font-semibold uppercase tracking-[0.3em] text-torea-bay-700 dark:text-slate-500">
						Result
					</h2>
					<span class="rounded-full border border-slate-200/60 bg-white px-3 py-1 text-xs font-semibold text-slate-500 shadow-sm dark:border-slate-700 dark:bg-slate-800/60 dark:text-slate-300">
						Realtime update
					</span>
				</div>

				<div class="space-y-6 rounded-3xl bg-gradient-to-br from-torea-bay-500 via-torea-bay-400 to-torea-bay-500 p-[1px]">
					<div class="flex flex-col gap-6 rounded-[calc(theme(spacing.6)*1.2)]  px-6 py-8 text-chateau-green-600 shadow-xl shadow-torea-bay-500/20 bg-torea-bay-50 dark:shadow-none">
						<div>
							<p class="text-sm uppercase tracking-[0.4em] text-torea-bay-950/90">Wet Bulb Target</p>
							<p class="mt-3 text-5xl font-black tracking-tight sm:text-6xl">
								{formatNumber(wetBulb)}<span class="ml-2 text-lg font-medium text-chateau-green-600/90">°C</span>
							</p>
						</div>

						<div class="grid gap-4 text-sm text-slate-400 sm:grid-cols-2">
							<div class="rounded-2xl border border-slate-100/80 bg-white px-4 py-3 ">
								<p class="text-xs uppercase tracking-[0.3em] text-torea-bay-950/90 ">
									Inputs
								</p>
								<p class="mt-2 font-medium text-torea-bay-600 ">
									{formatNumber(temperature)} °C
								</p>
								<p class="font-medium text-torea-bay-600 ">
									{formatNumber(humidity)} %
								</p>
							</div>

						</div>
					</div>
				</div>

				<div class="space-y-3 text-sm text-torea-bay-950/90 dark:text-slate-300">

					<p class="text-xs text-torea-bay-950/90 dark:text-slate-500">
						Formula based on the Stull approximation (2011). Accuracy ±1°C for 0°C ≤ T ≤ 50°C and 5% ≤ RH ≤ 99%.
					</p>
				</div>
			</section>
		</main>

		<footer class="mt-12 text-center text-xs text-torea-bay-950/90 dark:text-slate-500">
			<span>Designed by Edge Works · <a href="https://theedge.works" target="_blank">theedge.works</a></span>
		</footer>
	</div>
</div>
