<script lang="ts">
	import type { PeriodEntity } from '$lib/Attendance';
	import { attendance, attendanceLoaded } from '$lib/stores';
	import { loadAttendance } from '$lib/cache';
	import LoadingBanner from '$lib/LoadingBanner.svelte';
	import { Badge, Accordion, AccordionItem } from 'flowbite-svelte';
	import { fullDateFormatter, removeClassID } from '$lib';

	if (!$attendance) loadAttendance();

	function getAbsenceType(periods: PeriodEntity[]) {
		const reasons = periods.map((period: PeriodEntity) => period._Name);

		if (reasons.some((reason) => reason == 'Absent')) return 'Absent';

		if (reasons.some((reason) => reason.match(/Tardy/))) return 'Tardy';

		if (reasons.some((reason) => reason.match(/Field Trip|School Pass|Excused|Medical\/Dent/)))
			return 'Excused';

		return 'Unknown';
	}

	function getAbsenceColor(absenceType: string) {
		switch (absenceType) {
			case 'Absent':
				return 'red';
			case 'Tardy':
				return 'yellow';
			case 'Excused':
				return 'green';
			default:
				return 'primary';
		}
	}
</script>

<LoadingBanner show={!$attendanceLoaded} loadingMsg="Loading attendance..." />

{#if $attendance}
	<Accordion class="m-4">
		{#each $attendance.Absences.Absence ?? [] as absence}
			<AccordionItem class="dark:">
				<div slot="header">
					{fullDateFormatter.format(new Date(absence._AbsenceDate))}
					{#if getAbsenceType(absence.Periods.Period ?? [])}
						<Badge
							color={getAbsenceColor(getAbsenceType(absence.Periods.Period ?? []) ?? 'unknown')}
							large
						>
							{getAbsenceType(absence.Periods.Period ?? [])}
						</Badge>
					{/if}
				</div>
				<ol>
					{#each absence.Periods.Period?.filter((course) => course._Name) ?? [] as period}
						<li>{removeClassID(period._Course)}: {period._Name}</li>
					{/each}
				</ol>
			</AccordionItem>
		{/each}
	</Accordion>
{/if}
