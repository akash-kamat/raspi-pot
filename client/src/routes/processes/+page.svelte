<script>
    import { onMount, onDestroy } from 'svelte';
    import { socket } from '$lib/stores/socket';

    let processes = [];
    let unsubscribe;
    let sortKey = 'cpu';
    let sortOrder = -1; // -1 for desc, 1 for asc

    function toggleSort(key) {
        if (sortKey === key) {
            sortOrder *= -1;
        } else {
            sortKey = key;
            sortOrder = -1;
        }
    }

    $: sortedProcesses = [...processes].sort((a, b) => {
        let valA = a[sortKey];
        let valB = b[sortKey];
        
        if (typeof valA === 'string') {
            valA = valA.toLowerCase();
            valB = valB.toLowerCase();
        }
        
        if (valA < valB) return -1 * sortOrder;
        if (valA > valB) return 1 * sortOrder;
        return 0;
    });

    onMount(() => {
        unsubscribe = socket.subscribe($socket => {
            if ($socket) {
                $socket.emit('processes:watch', 3000);
                $socket.on('processes:data', (data) => {
                    processes = data;
                });
            }
        });
    });

    onDestroy(() => {
        if (unsubscribe) unsubscribe();
        if ($socket) {
            $socket.emit('processes:unwatch');
            $socket.off('processes:data');
        }
    });

    function killProcess(pid) {
        if (confirm(`Kill process ${pid}?`)) {
            if ($socket) $socket.emit('process:kill', pid);
        }
    }
</script>

<div class="page">
    <header>
        <div class="title-group">
            <h1>Process Manager</h1>
            <p class="subtitle">Real-time system process monitoring</p>
        </div>
        <div class="quick-stats">
            <div class="stat-badge">
                <span class="label">Total Tasks</span>
                <span class="value">{processes.length}</span>
            </div>
        </div>
    </header>

    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th on:click={() => toggleSort('pid')} class="sortable">
                        PID
                        {#if sortKey === 'pid'}
                            <span class="material-icons sort-icon">{sortOrder === 1 ? 'arrow_drop_up' : 'arrow_drop_down'}</span>
                        {/if}
                    </th>
                    <th on:click={() => toggleSort('name')} class="sortable">
                        Name
                        {#if sortKey === 'name'}
                            <span class="material-icons sort-icon">{sortOrder === 1 ? 'arrow_drop_up' : 'arrow_drop_down'}</span>
                        {/if}
                    </th>
                    <th on:click={() => toggleSort('cpu')} class="sortable">
                        CPU %
                        {#if sortKey === 'cpu'}
                            <span class="material-icons sort-icon">{sortOrder === 1 ? 'arrow_drop_up' : 'arrow_drop_down'}</span>
                        {/if}
                    </th>
                    <th on:click={() => toggleSort('mem')} class="sortable">
                        Mem %
                        {#if sortKey === 'mem'}
                            <span class="material-icons sort-icon">{sortOrder === 1 ? 'arrow_drop_up' : 'arrow_drop_down'}</span>
                        {/if}
                    </th>
                    <th on:click={() => toggleSort('user')} class="sortable">
                        User
                        {#if sortKey === 'user'}
                            <span class="material-icons sort-icon">{sortOrder === 1 ? 'arrow_drop_up' : 'arrow_drop_down'}</span>
                        {/if}
                    </th>
                    <th class="text-right">Actions</th>
                </tr>
            </thead>
            <tbody>
                {#each sortedProcesses as proc (proc.pid)}
                    <tr>
                        <td class="pid">#{proc.pid}</td>
                        <td class="name-cell">
                            <span class="material-icons proc-icon">terminal</span>
                            <span class="name">{proc.name}</span>
                        </td>
                        <td>
                            <div class="stat-bar">
                                <div class="bar-fill" style="width: {proc.cpu}%; background: #2b6cee"></div>
                                <span>{proc.cpu}%</span>
                            </div>
                        </td>
                        <td>
                            <div class="stat-bar">
                                <div class="bar-fill" style="width: {proc.mem}%; background: #06d6a0"></div>
                                <span>{proc.mem}%</span>
                            </div>
                        </td>
                        <td class="user">{proc.user}</td>
                        <td class="actions-cell">
                            <button class="btn-kill" on:click={() => killProcess(proc.pid)}>
                                <span class="material-icons">close</span>
                                Kill
                            </button>
                        </td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>

<style>
    .page { 
        padding: 2.5rem; 
        max-width: 1400px;
        margin: 0 auto;
    }

    header {
        display: flex;
        justify-content: space-between;
        align-items: flex-end;
        margin-bottom: 2.5rem;
    }

    h1 {
        margin: 0;
        font-size: 2rem;
        font-weight: 700;
        color: #f6f6f8;
    }

    .subtitle {
        margin: 0.25rem 0 0 0;
        color: #64748b;
        font-size: 0.95rem;
    }

    .stat-badge {
        background: #1a2233;
        padding: 0.5rem 1rem;
        border-radius: 0.75rem;
        border: 1px solid #243046;
        display: flex;
        flex-direction: column;
        align-items: flex-end;
    }
    .stat-badge .label { font-size: 0.7rem; color: #64748b; text-transform: uppercase; }
    .stat-badge .value { font-size: 1.1rem; font-weight: 600; color: #cbd5e1; }

    .table-container {
        background: #1a2233;
        border-radius: 1rem;
        overflow: hidden;
        border: 1px solid #243046;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        text-align: left;
    }

    th, td {
        padding: 1rem 1.25rem;
        border-bottom: 1px solid #243046;
    }

    th {
        background: #151c2c;
        color: #64748b;
        font-size: 0.8rem;
        text-transform: uppercase;
        letter-spacing: 1px;
        font-weight: 600;
        cursor: default;
    }

    th.sortable {
        cursor: pointer;
        user-select: none;
        transition: color 0.2s;
    }
    th.sortable:hover {
        color: #2b6cee;
    }

    .sort-icon {
        font-size: 1.2rem;
        vertical-align: middle;
        margin-left: 0.25rem;
    }

    .pid { color: #64748b; font-family: monospace; font-size: 0.9rem; }
    
    .name-cell {
        display: flex;
        align-items: center;
        gap: 0.75rem;
    }
    .proc-icon { color: #2b6cee; font-size: 1.1rem; }
    .name { font-weight: 600; color: #cbd5e1; }

    .stat-bar {
        display: flex;
        align-items: center;
        gap: 0.75rem;
        font-size: 0.85rem;
        color: #cbd5e1;
        width: 120px;
    }
    .bar-fill {
        height: 6px;
        border-radius: 3px;
        background: #243046;
    }

    .user { color: #64748b; font-size: 0.9rem; }

    .actions-cell {
        display: flex;
        justify-content: flex-end;
    }

    .btn-kill {
        display: inline-flex;
        align-items: center;
        gap: 0.3rem;
        background: rgba(239, 71, 111, 0.1);
        color: #ef476f;
        border: none;
        padding: 0.4rem 0.75rem;
        border-radius: 0.5rem;
        cursor: pointer;
        font-size: 0.8rem;
        transition: all 0.2s;
        font-weight: 600;
    }
    .btn-kill:hover { background: #ef476f; color: #f6f6f8; }
    .btn-kill .material-icons { font-size: 1rem; }
</style>
