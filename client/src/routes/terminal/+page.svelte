<script>
  import Terminal from '$lib/components/Terminal.svelte';
</script>

<div class="page">
    <header>
        <div class="title-group">
            <h1>System Terminal</h1>
            <p class="subtitle">Direct SSH access to raspberrypi.local</p>
        </div>
        <div class="status-indicator">
            <span class="dot"></span>
            Connected
        </div>
    </header>

    <div class="terminal-container">
        <Terminal />
    </div>
</div>

<style>
    .page { 
        padding: 2.5rem; 
        height: 100vh;
        display: flex;
        flex-direction: column;
        box-sizing: border-box;
    }

    header {
        display: flex;
        justify-content: space-between;
        align-items: flex-end;
        margin-bottom: 2rem;
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

    .status-indicator {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        padding: 0.4rem 0.75rem;
        background: rgba(6, 214, 160, 0.1);
        color: #06d6a0;
        border-radius: 0.5rem;
        font-size: 0.85rem;
        font-weight: 600;
    }

    .dot {
        width: 8px;
        height: 8px;
        background: #06d6a0;
        border-radius: 50%;
        box-shadow: 0 0 8px #06d6a0;
    }

    .terminal-container {
        flex: 1;
        background: #0a0f1a;
        border-radius: 1rem;
        border: 1px solid #243046;
        overflow: hidden;
        position: relative;
    }
</style>
