<script lang="ts">
  import { GoogleAnalytics } from '@beyonk/svelte-google-analytics';

  import Orb from '@app/Orb';

  let code = '';
  let output = '';

  const updateCode = ({
    currentTarget,
  }: Event) => {
    const {
      innerText,
    } = currentTarget as HTMLDivElement;
    code = innerText;
  };

  $: codeLines = code
    .replace(
      /(\<\/[a-z]+\>\<[a-z]+\<|\<[a-z]+\>|\<\/[a-z]+\>)/g,
      '\n'
    )
    .split('\n');

  $: cleanCode = codeLines.join('\n');

  const run = () => {
    try {
      output = eval(cleanCode);
    } catch (e) {
      output = e;
    }
  };

  const handleHotkeys = (
    e: KeyboardEvent
  ) => {
    switch (e.key) {
      case 'Tab': {
        e.preventDefault();
        e.stopPropagation();
        if (window.getSelection) {
          const selection = window.getSelection();
          if (
            selection &&
            selection.getRangeAt &&
            selection.rangeCount
          ) {
            const range = selection.getRangeAt(
              0
            );
            range.deleteContents();
            range.insertNode(
              document.createTextNode(
                '  '
              )
            );
            range.collapse();
          }
        }
        break;
      }

      case 'Enter': {
        if (!e.ctrlKey) break;
        run();
      }

      default: {
        // no-op
      }
    }
  };
</script>

<GoogleAnalytics
  properties={[
    import.meta.env
      .SNOWPACK_PUBLIC_GOOGLE_ANALYTICS_ID,
  ]}
/>

<Orb />

<header class="app-bar">
  <button
    on:click={run}
    class="button-run"
  >
    Run Code
  </button>
</header>

<main class="container">
  <div
    class="editor"
    contenteditable
    on:input={updateCode}
    on:keydown={handleHotkeys}
  >
    Type your JavaScript here...
  </div>
  <div class="console">
    {output}
  </div>
</main>

<style>
  .app-bar {
    height: 80px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .button-run {
    height: 40px;
    border: none;
    opacity: 0.8;
    color: white;
    appearance: none;
    border-radius: 20px;
    background: #008800;
  }

  .button-run:hover {
    opacity: 1;
  }

  .container {
    display: flex;
    height: 100vh;
    flex-direction: column;
  }

  .editor {
    height: 50%;
    flex-basis: 50%;
    white-space: pre;
    overflow-x: auto;
    border-radius: 8px;
    margin: 0.2em 0.5em;
    width: calc(100% - 1em);
    border: solid 1px black;
    padding: 0.5em 1em 0.5em 2em;
  }

  .console {
    width: 100%;
    flex-basis: 50%;
    margin: 0.2em 0.5em;
    padding: 0.5em 1em 0.5em 2em;
  }

  @media only screen and (min-width: 568px) {
    .container {
      flex-direction: row;
    }

    .editor {
      width: 50%;
      height: 100%;
    }

    .console {
      height: 100%;
    }
  }
</style>
