<script lang="ts">
  import { onMount } from 'svelte';
  import posthog from 'posthog-js';
  import 'prism-themes/themes/prism-vsc-dark-plus.css';
  import { mdiArrowTopRight, mdiDotsVertical, mdiGithub, mdiTwitter } from '@mdi/js';

  import {
    AppBar,
    AppLayout,
    Button,
    Icon,
    MenuButton,
    QuickSearch,
    ThemeInit,
    ThemeSelect,
    ThemeSwitch,
    Tooltip,
    // TODO: move/update
    settings,
  } from 'svelte-ux';

  import { entries } from '@layerstack/utils';
  import { lgScreen } from '@layerstack/svelte-stores';

  import NavMenu from './_NavMenu.svelte';

  import { dev } from '$app/environment';
  import { afterNavigate, goto } from '$app/navigation';
  import { page } from '$app/stores';

  import './app.css';

  export let data;

  const baseGh = 'https://github.com/techniq/layerstack';
  $: ghLink = data.pr_id ? `${baseGh}/pull/${data.pr_id}` : baseGh;
  const baseTitle = 'LayerStack';
  $: title = data.pr_id ? `🚧 (pr:${data.pr_id}) - ${baseTitle}` : baseTitle;

  settings({
    components: {
      AppLayout: {
        classes: {
          aside: 'border-r',
          nav: 'bg-surface-300 py-2',
        },
      },
      AppBar: {
        classes:
          'bg-primary text-primary-content shadow-md [text-shadow:1px_1px_2px_theme(colors.primary-700)]',
      },
      NavItem: {
        classes: {
          root: 'text-sm text-surface-content/70 pl-6 py-2 hover:bg-surface-100/70 relative',
          active:
            'text-primary bg-surface-100 font-medium before:absolute before:bg-primary before:rounded-full before:w-1 before:h-2/3 before:left-[6px] shadow z-10',
        },
      },
    },
    themes: data.themes,
  });

  let mainEl: HTMLElement;
  afterNavigate(() => {
    mainEl.scrollTo({ top: 0, behavior: 'instant' });
  });

  const groups = ['components', 'actions', 'stores', 'utils'];
  const quickSearchOptions = entries(
    import.meta.glob('./docs/**/+page.(md|svelte)', { query: '?raw', eager: true })
  )
    .flatMap(([file, source]) => {
      const url = file.replace('.', '').replace(/\/\+page.(md|svelte)/, '');
      const [_, docs, group, name] = url.split('/');
      return {
        label: name,
        value: url,
        group: group,
      };
    })
    .sort((a, b) => groups.indexOf(a.group) - groups.indexOf(b.group));

  let currentPath = '';
  onMount(() => {
    // Delay adding `scroll-smooth` to `<html>` as provides better refresh experience
    // and fixes issue where sometimes doesn't scroll far enough
    setTimeout(() => {
      document.documentElement.classList.add('scroll-smooth');
    }, 0);

    // Posthog analytics
    if (!dev) {
      const unsubscribePage = page.subscribe(($page) => {
        if (currentPath && currentPath !== $page.url.pathname) {
          // Page navigated away
          // @ts-expect-error - .capture() exists
          posthog.capture('$pageleave');
        }

        // Page entered
        currentPath = $page.url.pathname;
        // @ts-expect-error - .capture() exists
        posthog.capture('$pageview');
      });

      const handleBeforeUnload = () => {
        // Hard reloads or browser exit
        // @ts-expect-error - .capture() exists
        posthog.capture('$pageleave');
      };
      window.addEventListener('beforeunload', handleBeforeUnload);

      return () => {
        unsubscribePage();
        window.removeEventListener('beforeunload', handleBeforeUnload);
      };
    }
  });
</script>

<svelte:head>
  {#if $page.url.origin.includes('https')}
    <script
      defer
      src="https://static.cloudflareinsights.com/beacon.min.js"
      data-cf-beacon={JSON.stringify({ token: 'a9bb144411004657989216361774b0e7' })}
    ></script>

    <script
      async
      defer
      src="https://us.umami.is/script.js"
      data-website-id="e2c58ad6-4f28-4429-aecc-746e2cdf7a90"
    ></script>
  {/if}
</svelte:head>

<!-- Set theme before anything renders (even when SSR is in use) -->
<ThemeInit />

<AppLayout headerHeight={96}>
  <svelte:fragment slot="nav">
    <NavMenu />
    <!-- Spacer -->
    <div class="h-4"></div>
  </svelte:fragment>

  <AppBar {title} class="pt-8">
    <div
      class="fixed top-0 left-0 w-full h-8 bg-primary-700 border-b border-primary-800/50 shadow flex gap-2 items-center justify-center text-sm font-medium"
    >
      LayerStack for Svelte 5 has been released! -
      <a
        href="https://next.layerstack.dev"
        class="font-semibold underline-offset-2 hover:underline"
      >
        Visit the preview docs <Icon data={mdiArrowTopRight} class="text-xs mt-[-2px]" />
      </a>
    </div>

    <div slot="actions" class="flex gap-3">
      <Button
        href="https://www.layerchart.com"
        icon={{ data: mdiArrowTopRight, class: 'opacity-50' }}
        target="_blank"
        class="p-2 max-lg:hidden flex-row-reverse"
      >
        LayerChart
      </Button>

      <QuickSearch
        options={quickSearchOptions}
        on:change={(e) => {
          // @ts-expect-error
          goto(e.detail.value);
        }}
        classes={{ button: 'max-sm:-mr-3' }}
      />

      <div class="border-r border-primary-content/20 pr-2">
        {#if data.themes.light.length > 1 || data.themes.dark.length > 1}
          <ThemeSelect keyboardShortcuts />
        {:else}
          <ThemeSwitch classes={{ switch: 'bg-black/10' }} />
        {/if}
      </div>

      {#if $lgScreen}
        <Tooltip title="Discord" placement="left" offset={2}>
          <Button
            icon="M20.33 5.06C18.78 4.33 17.12 3.8 15.38 3.5 15.17 3.89 14.92 4.4 14.74 4.82 12.9 4.54 11.07 4.54 9.26 4.82 9.09 4.4 8.83 3.89 8.62 3.5 6.88 3.8 5.21 4.33 3.66 5.06 0.53 9.79-0.32 14.41 0.1 18.96 2.18 20.52 4.19 21.46 6.17 22.08 6.66 21.4 7.1 20.69 7.48 19.93 6.76 19.66 6.07 19.33 5.43 18.94 5.6 18.81 5.77 18.68 5.93 18.54 9.88 20.39 14.17 20.39 18.07 18.54 18.23 18.68 18.4 18.81 18.57 18.94 17.92 19.33 17.24 19.66 16.52 19.94 16.9 20.69 17.33 21.41 17.82 22.08 19.8 21.46 21.82 20.52 23.9 18.96 24.4 13.69 23.05 9.11 20.33 5.06ZM8.01 16.17C6.83 16.17 5.86 15.06 5.86 13.71 5.86 12.36 6.81 11.25 8.01 11.25 9.22 11.25 10.19 12.36 10.17 13.71 10.17 15.06 9.22 16.17 8.01 16.17ZM15.99 16.17C14.8 16.17 13.83 15.06 13.83 13.71 13.83 12.36 14.78 11.25 15.99 11.25 17.19 11.25 18.17 12.36 18.14 13.71 18.14 15.06 17.19 16.17 15.99 16.17Z"
            href="https://discord.gg/697JhMPD3t"
            class="p-2"
            target="_blank"
          />
        </Tooltip>

        <Tooltip title="Open Twitter / X" placement="left" offset={2}>
          <Button
            icon={mdiTwitter}
            href="https://twitter.com/techniq35"
            class="p-2"
            target="_blank"
          />
        </Tooltip>

        <Tooltip title="View repository" placement="left" offset={2}>
          <Button icon={mdiGithub} href={ghLink} class="p-2" target="_blank" />
        </Tooltip>
      {:else}
        <MenuButton
          icon={mdiDotsVertical}
          menuIcon={null}
          iconOnly={true}
          options={[
            {
              label: 'LayerChart',
              value: 'https://www.layerchart.com',
              icon: mdiArrowTopRight,
            },
            {
              label: 'Github',
              value: ghLink,
              icon: mdiGithub,
            },
            {
              label: 'Discord',
              value: 'https://discord.gg/697JhMPD3t',
              icon: 'M20.33 5.06C18.78 4.33 17.12 3.8 15.38 3.5 15.17 3.89 14.92 4.4 14.74 4.82 12.9 4.54 11.07 4.54 9.26 4.82 9.09 4.4 8.83 3.89 8.62 3.5 6.88 3.8 5.21 4.33 3.66 5.06 0.53 9.79-0.32 14.41 0.1 18.96 2.18 20.52 4.19 21.46 6.17 22.08 6.66 21.4 7.1 20.69 7.48 19.93 6.76 19.66 6.07 19.33 5.43 18.94 5.6 18.81 5.77 18.68 5.93 18.54 9.88 20.39 14.17 20.39 18.07 18.54 18.23 18.68 18.4 18.81 18.57 18.94 17.92 19.33 17.24 19.66 16.52 19.94 16.9 20.69 17.33 21.41 17.82 22.08 19.8 21.46 21.82 20.52 23.9 18.96 24.4 13.69 23.05 9.11 20.33 5.06ZM8.01 16.17C6.83 16.17 5.86 15.06 5.86 13.71 5.86 12.36 6.81 11.25 8.01 11.25 9.22 11.25 10.19 12.36 10.17 13.71 10.17 15.06 9.22 16.17 8.01 16.17ZM15.99 16.17C14.8 16.17 13.83 15.06 13.83 13.71 13.83 12.36 14.78 11.25 15.99 11.25 17.19 11.25 18.17 12.36 18.14 13.71 18.14 15.06 17.19 16.17 15.99 16.17Z',
            },
            {
              label: 'Twitter / X',
              value: 'https://twitter.com/techniq35',
              icon: mdiTwitter,
            },
          ]}
          on:change={(e) => {
            window.open(e.detail.value, '_blank');
          }}
        >
          <span slot="selection" class="hidden"></span>
        </MenuButton>
      {/if}
    </div>
  </AppBar>

  <main class="isolate" bind:this={mainEl}>
    <slot />
  </main>
</AppLayout>
