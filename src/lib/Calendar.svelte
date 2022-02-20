<script>
	import { fade } from 'svelte/transition';
	import moment from 'moment';

	moment.locale('en', {
		week: {
			dow: 6 // Saturday is the first day of the week.
		}
	});

	export let selected_day;
	let today = moment();

	$: m = moment(selected_day);
	$: month = m.format('MMMM');
	$: year = m.format('YYYY');
	$: month_start = m.clone().startOf('month');
	$: month_end = m.clone().endOf('month');
	$: cal_page_start = month_start.clone().startOf('week');
	$: cal_page_end = month_end.clone().endOf('week');
	$: cal_page_days = generateDaysBetween(cal_page_start, cal_page_start.clone().add(41, 'days'));
	$: cal_page_weeks = getWeekNumbers(cal_page_days);
	$: cal_day_details = getDayDetails(moment(selected_day));

	let loading = false;
	let reload_key = {};

	function reload() {
		reload_key = {};
	}

	function generateDaysBetween(start, end) {
		let days = [];
		let current = start.clone();
		while (current <= end) {
			days.push(current);
			current = current.clone().add(1, 'day');
		}
		return days;
	}

	function getWeekNumbers(days) {
		let weeks = [];
		for (let i = 0; i < days.length; i += 7) {
			weeks.push(days[i].format('W'));
		}
		return weeks;
	}

	function inCurrentMonth(day) {
		return day.isSame(m, 'month');
	}

	function isToday(day) {
		return day.isSame(today, 'day');
	}

	function isSelected(day) {
		return day.isSame(selected_day, 'day');
	}

	function navPreviousMonth() {
		m = m.clone().subtract(1, 'month');
		reload();
	}

	function navNextMonth() {
		m = m.clone().add(1, 'month');
		reload();
	}

	function navPreviousYear() {
		m = m.clone().subtract(1, 'year');
		reload();
	}

	function navNextYear() {
		m = m.clone().add(1, 'year');
		reload();
	}

	function navToday() {
		m = today.clone();
		reload();
	}

	function selectDay(day) {
		let do_reload = day.isSame(m, 'month') === false;
		selected_day = day.toDate();
		if (do_reload) {
			reload();
		}
	}

	async function getDayDetails(day) {
		if (day) {
			let month = day.format('M');
			let day_of_month = day.format('D');
			let history = await fetch(`https://history.muffinlabs.com/date/${month}/${day_of_month}`);
			return history.json();
		}
	}
</script>

<div class="calendar-page">
	<h1 class="page-header">{month} {year}</h1>

	<div class="calendar-nav-controls">
		<input type="button" value="<<" on:click={navPreviousYear} />
		<input type="button" value="<" on:click={navPreviousMonth} />
		<input type="button" value="today" on:click={navToday} />
		<input type="button" value=">" on:click={navNextMonth} />
		<input type="button" value=">>" on:click={navNextYear} />
	</div>

	<ul class="calendar-dow-headers">
		<li class="calendar-dow-header">Saturday</li>
		<li class="calendar-dow-header">Sunday</li>
		<li class="calendar-dow-header">Monday</li>
		<li class="calendar-dow-header">Tuesday</li>
		<li class="calendar-dow-header">Wednesday</li>
		<li class="calendar-dow-header">Thursday</li>
		<li class="calendar-dow-header">Friday</li>
	</ul>

	<ul class="calendar-week-headers">
		{#each cal_page_weeks as week}
			<li class="calendar-week-header">
				Week {week}
			</li>
		{/each}
	</ul>

	<ul class="calendar-days">
		{#each cal_page_days as day, i}
			{#key reload_key}
				<li
					in:fade={{ duration: 500, delay: i * 15 }}
					out:fade={{ duration: 0 }}
					class="calendar-day"
					class:in-current-month={inCurrentMonth(day)}
					class:is-today={isToday(day)}
					class:is-selected={isSelected(day)}
					on:click={selectDay(day)}
				>
					<span class="calendar-day-number">
						{day.format('DD')}
					</span>
				</li>
			{/key}
		{/each}
	</ul>

	<div class="day-details">
		{#await cal_day_details}
			fetching...
		{:then details}
			{#each details.data.Events as event}
				<p><b>Year {event.year}</b></p>
				<span>{event.text}</span>
			{/each}
		{/await}
	</div>
</div>

<style>
	:root {
		--calendar-day-width: 120px;
		--calendar-day-height: 120px;
	}

	.calendar-page {
		width: fit-content;
		display: grid;
		grid-template-areas:
			'filler       page-header'
			'filler       page-nav-controls'
			'filler       dow-headers'
			'week-headers calendar-days'
			'filler2      day-details';
	}

	.page-header {
		grid-area: page-header;
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 0;
		padding: 0;
	}

	.calendar-nav-controls {
		grid-area: page-nav-controls;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		margin: 0;
		padding: 0;
	}

	.calendar-nav-controls input {
		margin: 0 5px;
		background-color: Transparent;
		background-repeat: no-repeat;
		border: none;
		cursor: pointer;
		color: var(--color-highlight);
		font-size: 1rem;
		font-weight: bold;
	}

	.calendar-nav-controls input:hover {
		text-decoration: underline;
	}

	.calendar-dow-headers {
		grid-area: dow-headers;
		display: grid;
		gap: 0.5rem;
		grid-template-columns: repeat(7, var(--calendar-day-width));
		align-items: center;
		justify-content: center;
		margin: 0;
		padding: 0;
	}

	.calendar-dow-header {
		display: flex;
		flex-direction: column;
		justify-content: center;
		text-align: center;
		font-weight: bold;
		font-size: 1.2rem;
		padding: 0.5rem;
	}

	.calendar-week-headers {
		grid-area: week-headers;
		display: grid;
		width: 100px;
		margin: 0;
		padding: 0;
	}

	.calendar-week-header {
		display: flex;
		flex-direction: column;
		justify-content: center;
		text-align: center;
		font-weight: bold;
		font-size: 1.2rem;
		padding: 0.5rem;
	}

	.calendar-days {
		grid-area: calendar-days;
		display: grid;
		gap: 0.5rem;
		grid-template-columns: repeat(7, var(--calendar-day-width));
		grid-template-rows: repeat(6, var(--calendar-day-height));
		margin: 0;
		padding: 0;
	}

	.calendar-day {
		position: relative;
		width: 100%;
		height: 100%;
		list-style-type: none;
		display: block;
		background-color: var(--color-light);
		opacity: 0.5;
		border-radius: 5px;
		margin: 5px;
		user-select: none;
		margin: 0;
		padding: 0;
		cursor: pointer;
	}

	.in-current-month.calendar-day {
		opacity: 0.85;
	}

	.calendar-day:hover {
		opacity: 1;
	}

	.is-today.calendar-day {
		background-color: var(--color-highlight);
	}

	.is-selected.calendar-day .calendar-day-number {
		/* border: 3px solid var(--color-dark); */
		/* padding: 3px; */
		/* color: var(--color-light); */
		/* background-color: var(--color-dark); */
		/* border-radius: 50%; */
	}

	/* .is-selected.calendar-day {
        opacity: 1;
    } */

	.is-selected.calendar-day::before {
		content: '';
		position: absolute;
		background-color: var(--color-dark);
		top: 5px;
		left: 5px;
		width: 15px;
		height: 15px;
		border-radius: 5px;
	}

	.calendar-day-number {
		display: inline-block;
		width: fit-content;
		padding: 0.2rem;
		font-size: 0.7rem;
		font-weight: bold;
		position: absolute;
		top: 5px;
		right: 5px;
		color: rgba(0, 0, 0, 0.5);
	}

	.in-current-month .calendar-day-number {
		color: black;
	}

	.is-today .calendar-day-number {
		font-weight: bold;
		font-size: 1.1rem;
	}

	.day-details {
		grid-area: day-details;
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-items: flex-start;
		margin-top: 1rem;
		padding: 1rem;
		height: 100px;
		border: 1px solid var(--color-light);
		border-radius: 5px;
		overflow-y: scroll;
		width: 850px;
	}
</style>
