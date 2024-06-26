<script context="module" lang="ts">
	export function iso(date: Date) {
		const pad = (n: string | number) => ((n as number) < 10 ? '0' + n : n);
		return date.getFullYear() + '-' + pad(date.getMonth() + 1) + '-' + pad(date.getDate());
	}
</script>

<script lang="ts">
	import { writable } from 'svelte/store';

	export let value = '';
	export let days = 'S|M|T|W|T|F|S'.split('|');
	export let months = 'Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec'.split('|');
	export let start = 0; // first day of the week (0 = Sunday, 1 = Monday)
	export let offset = 0; // offset in months from currently selected date
	export let onOKClick = () => {};
	export let onCancelClick = () => {};

	let date = iso(new Date());
	let selected = writable(false);
	const today = iso(new Date());

	$: acceptDate(value);

	function acceptDate(value: string | number | Date) {
		const newDate = new Date(value);

		if (!isNaN(newDate as unknown as number)) {
			date = iso(newDate);
		}
	}

	function go(direction: number) {
		offset = offset + direction;
	}

	function selectDate(newValue: string) {
		date = newValue;
		value = newValue;
		offset = 0;
		selected.set(true);
		weeks = weeksFrom(viewDate, date, start);
	}

	$: viewDate = viewDateFrom(date, offset);

	$: month = months[viewDate.getMonth()];

	$: year = viewDate.getFullYear();

	$: weeks = weeksFrom(viewDate, date, start);

	function viewDateFrom(date: string | number | Date, offset: number) {
		var viewDate = new Date(date);
		viewDate.setMonth(viewDate.getMonth() + offset);
		return viewDate;
	}

	function weeksFrom(viewDate: Date, date: string, start: number) {
		var first = new Date(viewDate.getTime());
		first.setDate(1);
		first.setDate(first.getDate() + ((start - first.getDay() - 7) % 7));

		var last = new Date(viewDate.getTime());
		last.setDate(new Date(viewDate.getFullYear(), viewDate.getMonth() + 1, 0).getDate());
		last.setDate(last.getDate() + ((start - last.getDay() + 6) % 7));

		var d = new Date(first.getTime()),
			M = viewDate.getMonth(),
			Y = viewDate.getFullYear(),
			week: any[] = [],
			weeks = [week];

		while (d.getTime() <= last.getTime()) {
			var dd = d.getDate(),
				mm = d.getMonth(),
				yy = d.getFullYear(),
				value = iso(d);

			week.push({
				date: dd,
				value,
				class: [
					date === value && $selected ? 'selected' : '',
					mm == M ? '' : (mm > M ? yy >= Y : yy > Y) ? 'future' : 'past',
					today === value ? 'today' : ''
				].join(' ')
			});

			d = new Date(d.getFullYear(), d.getMonth(), d.getDate() + 1);

			if (d.getDay() === start) {
				week = [];
				weeks.push(week);
			}
		}
		weeks.pop();
		return weeks;
	}
</script>

<div class="absolute z-10 rounded-2xl bg-[color:var(--md-sys-color-surface-container-high)]">
	<div
		class="flex h-16 flex-row items-center justify-around text-[color:var(--md-sys-color-on-surface-variant)]"
	>
		<!-- svelte-ignore a11y-no-static-element-interactions -->
		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<div class="btn cursor-pointer" on:click={() => go(-1)}>
			<md-icon>chevron_left</md-icon>
		</div>
		<div class="font-medium">{month} {year}</div>
		<!-- svelte-ignore a11y-no-static-element-interactions -->
		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<div class="btn cursor-pointer" on:click={() => go(+1)}>
			<md-icon>chevron_right</md-icon>
		</div>
	</div>
	<div class="px-3 font-normal text-[color:var(--md-sys-color-on-surface)]">
		<div class="flex h-12 flex-row items-center justify-center">
			{#each days as day}
				<div class="flex h-full min-w-12 items-center justify-center">
					<span>{day}</span>
				</div>
			{/each}
		</div>
		{#each weeks as week}
			<div class="flex h-12 flex-row items-center justify-center">
				{#each week as day}
					<!-- svelte-ignore a11y-click-events-have-key-events -->
					<!-- svelte-ignore a11y-no-static-element-interactions -->
					<div
						class="btn cursor-pointer {day.class} relative flex h-full min-w-12 items-center justify-center"
						on:click={() => selectDate(day.value)}
					>
						<div class="absolute h-10 w-10 rounded-full"></div>
						<span class="z-10">{day.date}</span>
					</div>
				{/each}
			</div>
		{/each}
	</div>
	<div class="flex h-16 w-full flex-row items-center justify-end px-3 py-2">
		<div class="flex flex-row items-center justify-end gap-2">
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<md-text-button on:click={onCancelClick}>Cancel</md-text-button>
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<md-text-button on:click={onOKClick}>OK</md-text-button>
		</div>
	</div>
</div>

<style>
	.past span,
	.future span {
		color: var(--md-sys-color-outline);
		opacity: 0.8;
	}
	.today div {
		border-color: var(--md-sys-color-primary);
		border-width: 1px;
	}
	.today span {
		color: var(--md-sys-color-primary);
	}
	.selected div {
		background: var(--md-sys-color-primary) !important;
	}
	.selected span {
		color: var(--md-sys-color-on-primary);
	}
	.btn:hover div {
		background: rgba(100, 100, 100, 0.3);
	}
</style>
