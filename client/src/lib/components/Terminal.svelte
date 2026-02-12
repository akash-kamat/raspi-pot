<script>
  import { onMount, onDestroy } from 'svelte';
  import { Terminal } from 'xterm';
  import { FitAddon } from 'xterm-addon-fit';
  import { socket } from '$lib/stores/socket';
  import 'xterm/css/xterm.css';

  let terminalContainer;
  let term;
  let fitAddon;
  let unsubscribe;

  onMount(() => {
    // Initialize xterm.js
    term = new Terminal({
      cursorBlink: true,
      fontFamily: '"JetBrains Mono", "Cascadia Code", "Fira Code", monospace',
      fontSize: 14,
      theme: {
        background: '#0a0f1a',
        foreground: '#cbd5e1',
        cursor: '#2b6cee',
        selectionBackground: 'rgba(43, 108, 238, 0.3)',
        black: '#1e293b',
        red: '#ef476f',
        green: '#06d6a0',
        yellow: '#ffd166',
        blue: '#2b6cee',
        magenta: '#ae81ff',
        cyan: '#3a86ff',
        white: '#f6f6f8'
      }
    });

    fitAddon = new FitAddon();
    term.loadAddon(fitAddon);

    term.open(terminalContainer);
    fitAddon.fit();

    unsubscribe = socket.subscribe($socket => {
        if ($socket) {
            // Socket events
            if ($socket.connected) {
                onConnect($socket);
            }
            $socket.on('connect', () => onConnect($socket));
            $socket.on('terminal:output', (data) => {
                term.write(data);
            });
            $socket.on('disconnect', () => {
                term.write('\r\n*** Disconnected from backend ***\r\n');
            });
        }
    });

    function onConnect(s) {
      term.write('\r\n*** Connected to backend ***\r\n');
      const dims = { cols: term.cols, rows: term.rows };
      s.emit('terminal:start', dims);
    }

    // Terminal events
    term.onData((data) => {
      if ($socket) $socket.emit('terminal:input', data);
    });

    // Handle resize
    const handleResize = () => {
        fitAddon.fit();
        if ($socket && $socket.connected) {
            $socket.emit('terminal:resize', { cols: term.cols, rows: term.rows });
        }
    };
    window.addEventListener('resize', handleResize);

    // Initial focus
    term.focus();

    return () => {
      window.removeEventListener('resize', handleResize);
      if (unsubscribe) unsubscribe();
      if ($socket) {
          $socket.off('terminal:output');
          $socket.off('connect');
          $socket.off('disconnect');
      }
      if (term) term.dispose();
    };
  });
</script>

<div class="terminal-wrapper" bind:this={terminalContainer}></div>

<style>
  .terminal-wrapper {
    width: 100%;
    height: 100%;
    background: #0a0f1a;
    padding: 1rem;
    box-sizing: border-box;
    overflow: hidden;
  }
</style>
