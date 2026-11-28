<script lang="ts">
  import { Permissions } from '@rctf/types'
  import { useQueryClient } from '@tanstack/svelte-query'
  import wordmarkDark from '$lib/assets/wordmark-dark.svg'
  import wordmarkLight from '$lib/assets/wordmark-light.svg'
  import NavigationButton from '$lib/components/navigation-button.svelte'
  import NavigationCountdown from '$lib/components/navigation-countdown.svelte'
  import NavigationMobile from '$lib/components/navigation-mobile.svelte'
  import ThemeToggle from '$lib/components/theme-toggle.svelte'
  import {
    IconCopy,
    IconFlagBannerFold,
    IconGear,
    IconSignOut,
    IconTableFilled,
    IconUserGear,
  } from '$lib/icons'
  import { useClientConfig } from '$lib/query/config'
  import { useCurrentUser } from '$lib/query/user'
  import Avatar from '$lib/ui/avatar.svelte'
  import Button from '$lib/ui/button.svelte'
  import Menu from '$lib/ui/menu.svelte'
  import { copyLoginUrl, logout } from '$lib/utils/auth'
  import { countryCodeToFlagFilename } from '$lib/utils/flags'
  import {
    ADMIN_PANEL_PERMISSIONS,
    hasAnyPermission,
    hasPermissions,
  } from '$lib/utils/permissions'
  import { createRovingFocus } from '$lib/utils/roving'

  const rovingFocus = createRovingFocus()

  const queryClient = useQueryClient()
  const configQuery = useClientConfig()
  const clientConfig = $derived(configQuery.data)
  const userQuery = useCurrentUser()
  const user = $derived(userQuery.data)

  const isArchived = $derived(clientConfig?.isArchived ?? false)
  const canReadChallenges = $derived(
    hasPermissions(user, Permissions.challsRead)
  )
  const canManageUsers = $derived(hasPermissions(user, Permissions.usersWrite))
  const canManageSettings = $derived(
    hasPermissions(user, Permissions.settingsWrite)
  )
  const isAdmin = $derived(hasAnyPermission(user, ADMIN_PANEL_PERMISSIONS))

  const lightLogo = $derived(clientConfig?.logoLightUrl || wordmarkLight)
  const darkLogo = $derived(clientConfig?.logoDarkUrl || wordmarkDark)

  const divisionLabel = $derived(
    user?.division
      ? (clientConfig?.divisions[user.division] ?? user.division)
      : 'No Division'
  )

  const adminMenuItems = $derived(
    [
      {
        value: 'admin-challenges',
        label: 'Manage challenges',
        icon: IconFlagBannerFold,
        href: '/admin/challenges',
        show: canReadChallenges,
      },
      {
        value: 'admin-teams',
        label: 'Manage teams',
        icon: IconUserGear,
        href: '/admin/teams',
        show: canManageUsers,
      },
      {
        value: 'admin-submissions',
        label: 'Submissions',
        icon: IconTableFilled,
        href: '/admin/submissions',
        show: canReadChallenges && canManageUsers,
      },
      {
        value: 'admin-settings',
        label: 'Settings',
        icon: IconGear,
        href: '/admin/settings',
        show: canManageSettings || canManageUsers,
      },
    ].filter(item => item.show)
  )

  const userMenuItems = $derived([
    {
      value: 'copy-login-url',
      label: 'Copy login URL',
      icon: IconCopy,
      onSelect: () => {
        if (user?.teamToken) copyLoginUrl(user.teamToken)
      },
    },
    {
      value: 'profile',
      label: 'Manage team',
      icon: IconUserGear,
      href: '/profile',
    },
    {
      value: 'logout',
      label: 'Log out',
      icon: IconSignOut,
      onSelect: () => logout(queryClient),
    },
  ])
</script>

<header>
  <header-start>
    <NavigationCountdown />
  </header-start>

  <nav aria-label="Main" {@attach rovingFocus}>
    <nav-group data-side="start">
      <NavigationButton
        data-roving
        href="/challenges"
        activePath="/challenges"
        label="Challenges"
      />
    </nav-group>

    <a href="/" aria-label="Home" data-slot="logo">
      <logo-light data-theme-visible="light"
        ><img src={lightLogo} alt={clientConfig?.ctfName} /></logo-light
      >
      <logo-dark data-theme-visible="dark"
        ><img src={darkLogo} alt={clientConfig?.ctfName} /></logo-dark
      >
    </a>

    <nav-group data-side="end">
      <NavigationButton
        data-roving
        href="/scores"
        activePath="/scores"
        label="Scoreboard"
      />
      {#if isAdmin}
        <Menu label="Admin menu" items={adminMenuItems}>
          {#snippet trigger({ props })}
            <NavigationButton
              {...props}
              data-roving
              activePath="/admin"
              label="Admin"
              caret
            />
          {/snippet}
        </Menu>
      {/if}
    </nav-group>
  </nav>

  <header-end>
    {#if user && !isArchived}
      {@const currentUser = user}
      {#await import('$lib/components/navigation-team-stats.svelte') then { default: NavigationTeamStats }}
        <NavigationTeamStats />
      {/await}
      <Menu label="Account" items={userMenuItems} placement="bottom-end">
        {#snippet trigger({ props })}
          <button {...props} data-slot="account">
            <user-details>
              <user-name>{currentUser.name}</user-name>
              <user-meta>
                {#if currentUser.countryCode}
                  <img
                    src="/flags/{countryCodeToFlagFilename(
                      currentUser.countryCode
                    )}"
                    alt="{currentUser.countryCode} flag"
                  />
                {/if}
                {#if currentUser.countryCode && currentUser.statusText}
                  <meta-separator>·</meta-separator>
                {/if}
                {#if currentUser.statusText}
                  <user-status>{currentUser.statusText}</user-status>
                {:else if !currentUser.countryCode}
                  <user-status>{divisionLabel}</user-status>
                {/if}
              </user-meta>
            </user-details>
            <Avatar src={currentUser.avatarUrl} name={currentUser.name} />
          </button>
        {/snippet}
      </Menu>
    {:else if !isArchived}
      <a href="/login" data-slot="login">Log in</a>
      {#if clientConfig?.registrationsEnabled !== false}
        <Button href="/register">Register</Button>
      {/if}
    {/if}
    <ThemeToggle />
    <NavigationMobile />
  </header-end>
</header>

<style>
  header {
    position: fixed;
    inset-block-start: 0;
    inset-inline: 0;
    z-index: var(--layer-nav);
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    align-items: center;
    gap: var(--space-s);
    block-size: var(--header-height);
    padding-inline: 1rem;
    background: var(--background-l0);

    @media (width >= 48rem) {
      padding-inline: 1.25rem;
    }
  }

  header-start {
    display: flex;
    align-items: center;
  }

  nav {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    align-items: center;
    gap: var(--space-l);
  }

  nav-group {
    display: none;
    align-items: center;
    gap: var(--space-l);

    &[data-side='start'] {
      justify-content: flex-end;
    }

    @media (width >= 48rem) {
      display: flex;
    }
  }

  [data-slot='logo'] {
    grid-column: 2;
    display: block;

    img {
      display: block;
      block-size: 2.5rem;
      inline-size: auto;
      max-inline-size: 12rem;
    }
  }

  header-end {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: var(--space-xs);

    > :global(:not(mobile-nav)) {
      @media (width < 48rem) {
        display: none;
      }
    }
  }

  [data-slot='login'] {
    --underline: transparent;

    font-size: 0.875rem;
    letter-spacing: 0.025em;
    color: var(--foreground-l0);
    white-space: nowrap;
    transition: opacity 120ms ease;

    &:hover {
      opacity: 0.6;
    }
  }

  [data-slot='account'] {
    --avatar-size: 2.25rem;

    display: flex;
    align-items: center;
    gap: 0.625rem;
    padding: 0.25rem;
    cursor: pointer;
    border-radius: var(--radius-full);

    @media (width >= 72rem) {
      padding-inline-start: 0.75rem;
    }

    &:hover {
      background: var(--background-l2);
    }

    @media (width < 48rem) {
      display: none;
    }
  }

  user-details {
    display: none;
    flex-direction: column;
    align-items: end;

    @media (width >= 72rem) {
      display: flex;
    }
  }

  user-name {
    display: block;
    max-inline-size: 9rem;
    overflow: hidden;
    font-size: 0.875rem;
    font-weight: 400;
    line-height: 1.25;
    color: var(--foreground-l0);
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  user-meta {
    display: flex;
    align-items: center;
    gap: var(--space-3xs);

    img {
      block-size: 0.875rem;
      inline-size: auto;
      flex-shrink: 0;
    }
  }

  meta-separator {
    display: block;
    font-size: 0.875rem;
    line-height: 1;
    color: var(--foreground-l4);
  }

  user-status {
    display: block;
    max-inline-size: 8rem;
    overflow: hidden;
    font-size: 0.75rem;
    color: var(--foreground-l3);
    text-overflow: ellipsis;
    white-space: nowrap;
  }
</style>
