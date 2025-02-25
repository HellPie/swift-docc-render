<!--
  This source file is part of the Swift.org open source project

  Copyright (c) 2021-2024 Apple Inc. and the Swift project authors
  Licensed under Apache License v2.0 with Runtime Library Exception

  See https://swift.org/LICENSE.txt for license information
  See https://swift.org/CONTRIBUTORS.txt for Swift project authors
-->

<template>
  <NavBase
    :breakpoint="BreakpointName.medium"
    :hasOverlay="false"
    :hasNoBorder="hasNoBorder"
    :isDark="isDark"
    isWideFormat
    hasFullWidthBorder
    class="documentation-nav"
    :aria-label="$t('api-reference')"
    :showActions="hasMenuItems"
  >
    <template #pre-title="{ closeNav, isOpen, currentBreakpoint, className }" v-if="displaySidenav">
      <div :class="className">
        <div class="sidenav-toggle-wrapper">
          <button
            :aria-label="$t('navigator.open-navigator')"
            :id="baseNavOpenSidenavButtonId"
            class="sidenav-toggle"
            :tabindex="isOpen ? -1 : null"
            @click.prevent="handleSidenavToggle(closeNav, currentBreakpoint)"
          >
          <span class="sidenav-icon-wrapper">
            <SidenavIcon class="icon-inline sidenav-icon" />
          </span>
          </button>
        </div>
      </div>
    </template>
    <template #default>
      <slot name="title" className="nav-title" />
    </template>
    <template #tray="{ closeNav }">
      <NavMenuItems
        v-if="hasMenuItems"
        class="nav-menu-settings"
      >
        <LanguageToggle
          v-if="hasLanguageToggle"
          :interfaceLanguage="interfaceLanguage"
          :objcPath="objcPath"
          :swiftPath="swiftPath"
          :closeNav="closeNav"
        />
        <slot name="menu-items" />
      </NavMenuItems>
      <slot name="tray-after" />
    </template>
    <template #after-content>
      <slot name="after-content" />
    </template>
  </NavBase>
</template>

<script>
import NavBase from 'docc-render/components/NavBase.vue';
import NavMenuItems from 'docc-render/components/NavMenuItems.vue';
import { BreakpointName } from 'docc-render/utils/breakpoints';
import SidenavIcon from 'theme/components/Icons/SidenavIcon.vue';
import { SIDEBAR_HIDE_BUTTON_ID } from 'docc-render/constants/sidebar';
import { baseNavOpenSidenavButtonId } from 'docc-render/constants/nav';
import LanguageToggle from './DocumentationNav/LanguageToggle.vue';

export default {
  name: 'DocumentationNav',
  components: {
    SidenavIcon,
    NavBase,
    NavMenuItems,
    LanguageToggle,
  },
  props: {
    isDark: {
      type: Boolean,
      default: false,
    },
    hasNoBorder: {
      type: Boolean,
      default: false,
    },
    interfaceLanguage: {
      type: String,
      required: false,
    },
    objcPath: {
      type: String,
      required: false,
    },
    swiftPath: {
      type: String,
      required: false,
    },
    displaySidenav: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    baseNavOpenSidenavButtonId: () => baseNavOpenSidenavButtonId,
    BreakpointName: () => BreakpointName,
    hasLanguageToggle: ({ interfaceLanguage, swiftPath, objcPath }) => (
      !!(interfaceLanguage && (swiftPath || objcPath))),
    hasMenuItems: ({ hasLanguageToggle, $slots }) => !!(hasLanguageToggle || $slots['menu-items']),
  },
  methods: {
    async handleSidenavToggle(closeNav, currentBreakpoint) {
      // close the navigation
      await closeNav();
      // toggle the sidenav
      this.$emit('toggle-sidenav', currentBreakpoint);
      await this.$nextTick();
      const trigger = document.getElementById(SIDEBAR_HIDE_BUTTON_ID);
      if (trigger) {
        trigger.focus();
      }
    },
  },
};
</script>

<style scoped lang="scss">
@import 'docc-render/styles/_core.scss';

$sidenav-icon-size: 19px;
$sidenav-icon-padding-size: 5px;

// overwrite the typography of menu items outside of breakpoint only
:deep() .nav-menu {
  @include font-styles(documentation-nav);

  &-settings {
    // ensure settings can get smaller if needed
    min-width: 0;
    gap: $nav-space-between-elements;

    @include font-styles(nav-toggles);

    @include breakpoint-only-largenav() {
      margin-left: $nav-space-between-elements;
    }

    .nav-menu-setting {
      display: flex;
      align-items: center;
      color: var(--color-nav-current-link);
      margin-left: 0;
      min-width: 0;

      .nav-menu-link {
        font-weight: $font-weight-semibold;
        @include underline-text;
      }

      @include nav-dark() {
        color: var(--color-nav-dark-current-link);
      }

      @include nav-in-breakpoint() {
        &:not(:first-child) {
          border-top: 1px solid var(--color-fill-gray-tertiary);
        }
      }
    }
  }
}

.documentation-nav {
  --color-nav-background: var(--color-fill);

  :deep() {
    .nav-title {
      @include font-styles(nav-title-large);
    }
  }
}

.sidenav-toggle-wrapper {
  display: flex;
  margin-top: 1px;
  margin-right: $nav-padding / 2;

  // This is a hack to enforce the toggle to be visible when in breakpoint,
  // even if already toggled off on desktop. Conditionally checking the current breakpoint,
  // would trigger animations when switching between breakpoints.
  @include nav-in-breakpoint() {
    display: flex !important;
  }
}

// desktop only animation for the toggle
@include breakpoints-from(large, nav) {
  .sidenav-toggle-enter-active, .sidenav-toggle-leave-active {
    transition: margin $adjustable-sidebar-hide-transition-duration ease-in 0s;
  }
  .sidenav-toggle-enter, .sidenav-toggle-leave-to {
    // 2x the nav padding, 1px border, and the size of the icon
    margin-left: (rem($sidenav-icon-size + 1px) + $nav-padding * 2) * -1;
  }
}

.sidenav-toggle {
  align-self: center;
  color: var(--color-nav-link-color);
  position: relative;
  margin: 0 (-$sidenav-icon-padding-size);
  border-radius: $nano-border-radius;

  @include nav-dark {
    color: var(--color-nav-dark-link-color);
  }

  &:hover .sidenav-icon-wrapper {
    background: var(--color-fill-gray-quaternary);

    .theme-dark & {
      background: dark-color(fill-gray-quaternary);
    }
  }

  &__separator {
    height: .8em;
    width: 1px;
    background: var(--color-nav-color);
    align-self: center;
    margin: 0 $nav-padding;
  }

  @include nav-in-breakpoint() {
    &__separator {
      display: none;
    }
  }
}

.sidenav-icon-wrapper {
  padding: $sidenav-icon-padding-size;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: $nano-border-radius;
}

.sidenav-icon {
  display: flex;
  width: $sidenav-icon-size;
  height: $sidenav-icon-size;
}
</style>
