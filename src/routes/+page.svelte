<script>
		import Header from '../components/Header.svelte';
		import Lane from '../components/Lane.svelte';
		import IssueDialog from '../components/IssueDialog.svelte';
		import { v4 as uuidv4 } from 'uuid';
		import { formatISO } from 'date-fns';

		const LANE_ORDER = [
			{ id: 'todo', title: 'To Do' },
			{ id: 'doing', title: 'Doing' },
			{ id: 'done', title: 'Done' },
			{ id: 'archive', title: 'Archive' }
		];

		let lanes = $state(LANE_ORDER);
		let issues = $state([]);
		let dialogOpen = $state(false);
		let editingIssue = $state(null);
		const STORAGE_KEY = 'kanban_v1';

		function loadState() {
			try {
				const raw = localStorage.getItem(STORAGE_KEY);
				if (raw) {
					const parsed = JSON.parse(raw);
					issues = parsed.issues || [];
				}
			} catch (e) {
				console.error('load error', e);
				issues = [];
			}
		}

		function saveState() {
			localStorage.setItem(STORAGE_KEY, JSON.stringify({ issues }));
		}

		$effect(() => {
			loadState();
			if ('serviceWorker' in navigator) {
				navigator.serviceWorker.register('/service-worker.js').catch(console.error);
			}
		});

		function openCreate() {
			console.log('+page: openCreate triggered');
			editingIssue = null;
			dialogOpen = true;
		}

		function onCreate(detail) {
			console.log('+page: onCreate triggered', detail);
			const f = detail;
			const newIssue = {
				id: uuidv4(),
				title: f.title || 'Untitled',
				description: f.description || '',
				createdAt: formatISO(new Date()),
				dueDate: f.dueDate || '',
				storyPoints: Number(f.storyPoints || 1),
				priority: f.priority || 'Medium',
				laneId: 'todo'
			};
			issues = [newIssue, ...issues];
			saveState();
		}

		function onUpdate(detail) {
			console.log('+page: onUpdate triggered', detail);
			const f = detail;
			issues = issues.map((i) => (i.id === f.id ? { ...i, ...f } : i));
			saveState();
		}

		function onEdit(issue) {
			console.log('+page: onEdit triggered', issue);
			editingIssue = issue;
			dialogOpen = true;
		}

		function onDelete(id) {
			console.log('+page: onDelete triggered', id);
			issues = issues.filter((i) => i.id !== id);
			saveState();
		}

		async function onShare(id) {
			console.log('+page: onShare triggered', id);
			const i = issues.find((x) => x.id === id);
			if (!i) return;
			const text = `${i.title}\n\n${i.description}\n\nDue: ${i.dueDate || '—'}`;
			if (navigator.share) {
				try {
					await navigator.share({ title: i.title, text });
				} catch (err) {
					console.warn('share cancelled', err);
				}
			} else {
				await navigator.clipboard.writeText(text).catch(() => {});
				alert('Share not available — copied to clipboard.');
			}
		}

		function exportIssueICS(id) {
			console.log('+page: exportIssueICS triggered', id);
			const i = issues.find((x) => x.id === id);
			if (!i) return;
			const dtstart = i.dueDate
				? new Date(i.dueDate).toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z'
				: new Date().toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z';
			const uid = `${i.id}@kanban`;
			const ics = [
				'BEGIN:VCALENDAR',
				'VERSION:2.0',
				'PRODID:-//KanbanBoard//EN',
				'BEGIN:VEVENT',
				`UID:${uid}`,
				`DTSTAMP:${new Date().toISOString().replace(/[-:]/g, '').split('.')[0]}Z`,
				`DTSTART:${dtstart}`,
				`SUMMARY:${escapeIcs(i.title)}`,
				`DESCRIPTION:${escapeIcs(i.description || '')}`,
				'END:VEVENT',
				'END:VCALENDAR'
			].join('\r\n');
			const blob = new Blob([ics], { type: 'text/calendar' });
			const url = URL.createObjectURL(blob);
			const a = document.createElement('a');
			a.href = url;
			a.download = `${i.title || 'issue'}.ics`;
			a.click();
			URL.revokeObjectURL(url);
		}

		function escapeIcs(str = '') {
			return str.replace(/\n/g, '\\n').replace(/,/g, '\\,');
		}

		function exportCSV() {
			console.log('+page: exportCSV triggered');
			const headers = ['id', 'title', 'description', 'createdAt', 'dueDate', 'storyPoints', 'priority', 'laneId'];
			const rows = issues.map((i) => headers.map((h) => `"${String(i[h] ?? '').replace(/"/g, '""')}"`).join(','));
			const csv = [headers.join(','), ...rows].join('\r\n');
			const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
			const url = URL.createObjectURL(blob);
			const a = document.createElement('a');
			a.href = url;
			a.download = 'kanban-issues.csv';
			a.click();
			URL.revokeObjectURL(url);
		}

		function onDragStart(payload) {
			console.log('+page: onDragStart triggered', payload);
		}

		function onDrop(targetLaneId, e) {
			console.log('+page: onDrop triggered', targetLaneId, e);
			const id = e.dataTransfer.getData('text/issue-id');
			if (!id) return;
			const issue = issues.find((i) => i.id === id);
			if (!issue) return;
			const prevLane = issue.laneId;
			issues = issues.map((i) => (i.id === id ? { ...i, laneId: targetLaneId } : i));
			saveState();
			if (targetLaneId === 'done' && prevLane !== 'done') {
				notify(`Issue "${issue.title}" moved to Done.`);
			}
		}

		async function notify(message) {
			console.log('+page: notify triggered', message);
			if (!('Notification' in window)) return;
			if (Notification.permission === 'granted') {
				new Notification(message);
				return;
			}
			if (Notification.permission !== 'denied') {
				const perm = await Notification.requestPermission();
				if (perm === 'granted') new Notification(message);
			}
		}
</script>

<div class="min-h-screen bg-gray-50 dark:bg-gray-900 flex flex-col">
	<Header onnew={openCreate} onexportcsv={exportCSV} />
	<main class="flex-1 p-4 overflow-x-auto">
		<div class="flex gap-3 max-w-[1400px] mx-auto">
			{#each lanes as lane}
				<Lane
					{lane}
					issues={issues.filter((i) => i.laneId === lane.id)}
					ondrop={(e) => onDrop(lane.id, e)}
					ondragstart={onDragStart}
					onedit={onEdit}
					ondelete={onDelete}
					onshare={onShare}
					onexportics={exportIssueICS}
				/>
			{/each}
		</div>
	</main>
	<IssueDialog
		open={dialogOpen}
		issue={editingIssue}
		oncreate={onCreate}
		onupdate={onUpdate}
		onclose={() => {
			console.log('+page: IssueDialog onclose triggered');
			dialogOpen = false;
			editingIssue = null;
		}}
	/>
</div>