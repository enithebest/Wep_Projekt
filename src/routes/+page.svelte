<script>
  import Header from '../components/Header.svelte';
  import Lane from '../components/Lane.svelte';
  import IssueDialog from '../components/IssueDialog.svelte';
  import { onMount } from 'svelte';
  import { v4 as uuidv4 } from 'uuid';
  import { formatISO } from 'date-fns';

  const LANE_ORDER = [
    { id: 'todo', title: 'To Do' },
    { id: 'doing', title: 'Doing' },
    { id: 'done', title: 'Done' },
    { id: 'archive', title: 'Archive' }
  ];

  let lanes = LANE_ORDER;
  let issues = [];
  let dialogOpen = false;
  let editingIssue = null;

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

  onMount(() => {
    loadState();
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/service-worker.js').catch(console.error);
    }
  });

  function openCreate() {
    editingIssue = null;
    dialogOpen = true;
  }

  function onCreate(e) {
    const f = e.detail || e;
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

  function onUpdate(e) {
    const f = e.detail || e;
    issues = issues.map(i => (i.id === f.id ? { ...i, ...f } : i));
    saveState();
  }

  function onEdit(issue) {
    editingIssue = issue;
    dialogOpen = true;
  }

  function onDelete(id) {
    issues = issues.filter(i => i.id !== id);
    saveState();
  }

  async function onShare(id) {
    const i = issues.find(x => x.id === id);
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
    const i = issues.find(x => x.id === id);
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
    const headers = ['id', 'title', 'description', 'createdAt', 'dueDate', 'storyPoints', 'priority', 'laneId'];
    const rows = issues.map(i =>
      headers.map(h => `"${String(i[h] ?? '').replace(/"/g, '""')}"`).join(',')
    );
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
    // No state tracking needed
  }

  function onDrop(targetLaneId, e) {
    const id = e.dataTransfer.getData('text/issue-id');
    if (!id) return;
    const issue = issues.find(i => i.id === id);
    if (!issue) return;
    const prevLane = issue.laneId;
    issues = issues.map(i => (i.id === id ? { ...i, laneId: targetLaneId } : i));
    saveState();

    if (targetLaneId === 'done' && prevLane !== 'done') {
      notify(`Issue "${issue.title}" moved to Done.`);
    }
  }


  async function notify(message) {
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

<Header onnew={openCreate} onexportcsv={exportCSV} />

<main class="p-6 overflow-auto">
  <div class="flex gap-4">
    {#each lanes as lane}
      <Lane
        {lane}
        issues={issues.filter(i => i.laneId === lane.id)}
        onDrop={(e) => onDrop(lane.id, e.detail ? e.detail.event : e)}
        onDragStart={(e) => onDragStart(e.detail || e)}
        onEdit={(e) => onEdit(e.detail)}
        onDelete={(e) => onDelete(e.detail)}
        onShare={(e) => onShare(e.detail)}
        onExportICS={(e) => exportIssueICS(e.detail)}
      />
    {/each}
  </div>
</main>

<IssueDialog
  bind:open={dialogOpen}
  {editingIssue}
  issue={editingIssue}
  oncreate={(e) => onCreate(e)}
  onupdate={(e) => onUpdate(e)}
  onclose={() => {
    dialogOpen = false;
    editingIssue = null;
  }}
/>

<style>
  :global(body) {
    margin: 0;
    font-family: Arial, sans-serif;
  }
</style>