<script lang="ts">
  import type { ComboBoxOption } from '$lib/components/shared-components/combobox.svelte';
  import SettingCombobox from '$lib/components/shared-components/settings/setting-combobox.svelte';
  import SettingSwitch from '$lib/components/shared-components/settings/setting-switch.svelte';
  import SettingsLanguageSelector from '$lib/components/shared-components/settings/settings-language-selector.svelte';
  import { fallbackLocale, locales } from '$lib/constants';
  import { themeManager } from '$lib/managers/theme-manager.svelte';
  import {
    alwaysLoadOriginalFile,
    locale,
    loopVideo,
    playVideoThumbnailOnHover,
    showDeleteModal,
  } from '$lib/stores/preferences.store';
  import { findLocale } from '$lib/utils';
  import { onMount } from 'svelte';
  import { t } from 'svelte-i18n';
  import { fade } from 'svelte/transition';

  let time = $state(new Date());

  onMount(() => {
    const interval = setInterval(() => {
      time = new Date();
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  });

  const getAllLanguages = (): ComboBoxOption[] => {
    return locales
      .filter(({ code }) => Intl.NumberFormat.supportedLocalesOf(code).length > 0)
      .map((locale) => ({
        label: locale.name,
        value: locale.code,
      }));
  };

  const handleToggleLocaleBrowser = () => {
    $locale = $locale ? undefined : fallbackLocale.code;
  };

  const handleLocaleChange = (newLocale: string | undefined) => {
    if (newLocale) {
      $locale = newLocale;
    }
  };
  let editedLocale = $derived(findLocale($locale).code);
  let formattedDate = $derived(
    time.toLocaleString(editedLocale, {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
    }),
  );
  let timePortion = $derived(
    time.toLocaleString(editedLocale, {
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
    }),
  );
  let selectedDate = $derived(`${formattedDate} ${timePortion}`);
  let selectedOption = $derived({
    value: findLocale(editedLocale).code || fallbackLocale.code,
    label: findLocale(editedLocale).name || fallbackLocale.name,
  });
</script>

<section class="my-4">
  <div in:fade={{ duration: 500 }}>
    <div class="ms-4 mt-4 flex flex-col gap-4">
      <div class="ms-4">
        <SettingSwitch
          title={$t('theme_selection')}
          subtitle={$t('theme_selection_description')}
          checked={themeManager.theme.system}
          onToggle={(isChecked) => themeManager.setSystem(isChecked)}
        />
      </div>

      <div class="ms-4">
        <SettingsLanguageSelector showSettingDescription />
      </div>

      <div class="ms-4">
        <SettingSwitch
          title={$t('default_locale')}
          subtitle={$t('default_locale_description')}
          checked={$locale == undefined}
          onToggle={handleToggleLocaleBrowser}
        >
          <p class="mt-2 dark:text-gray-400">{selectedDate}</p>
        </SettingSwitch>
      </div>
      {#if $locale !== undefined}
        <div class="ms-4">
          <SettingCombobox
            comboboxPlaceholder={$t('searching_locales')}
            {selectedOption}
            options={getAllLanguages()}
            title={$t('custom_locale')}
            subtitle={$t('custom_locale_description')}
            onSelect={(combobox) => handleLocaleChange(combobox?.value)}
          />
        </div>
      {/if}

      <div class="ms-4">
        <SettingSwitch
          title={$t('display_original_photos')}
          subtitle={$t('display_original_photos_setting_description')}
          bind:checked={$alwaysLoadOriginalFile}
          onToggle={() => ($alwaysLoadOriginalFile = !$alwaysLoadOriginalFile)}
        />
      </div>
      <div class="ms-4">
        <SettingSwitch
          title={$t('video_hover_setting')}
          subtitle={$t('video_hover_setting_description')}
          bind:checked={$playVideoThumbnailOnHover}
          onToggle={() => ($playVideoThumbnailOnHover = !$playVideoThumbnailOnHover)}
        />
      </div>
      <div class="ms-4">
        <SettingSwitch
          title={$t('loop_videos')}
          subtitle={$t('loop_videos_description')}
          bind:checked={$loopVideo}
          onToggle={() => ($loopVideo = !$loopVideo)}
        />
      </div>

      <div class="ms-4">
        <SettingSwitch
          title={$t('permanent_deletion_warning')}
          subtitle={$t('permanent_deletion_warning_setting_description')}
          bind:checked={$showDeleteModal}
        />
      </div>
    </div>
  </div>
</section>
