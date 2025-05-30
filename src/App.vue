<template>
  <v-app>
    <v-main>
      <div
        v-if="
          !widgetStore.editingMode &&
          !interfaceStore.isMainMenuVisible &&
          interfaceStore.mainMenuStyleTrigger === 'center-left'
        "
      >
        <div
          id="menu-trigger"
          class="menu-trigger border-4 flex items-center justify-center w-[30px] px-0 py-2 cursor-pointer overflow-hidden rounded-r-lg rounded-br-lg -ml-[1px]"
          :class="[interfaceStore.isOnSmallScreen ? 'top-[30%]' : 'top-[50%]']"
          :style="
            interfaceStore.highlightedComponent === 'menu-trigger'
              ? interfaceStore.globalGlassMenuHighlightStyles
              : interfaceStore.globalGlassMenuStyles
          "
          @click="toggleMainMenu"
        >
          <v-icon class="text-white text-[46px] opacity-80">mdi-menu-right</v-icon>
        </div>
      </div>
      <transition name="slide-in-left">
        <div
          v-if="interfaceStore.isMainMenuVisible"
          ref="mainMenu"
          class="left-menu slide-in"
          :style="[
            glassMenuStyles,
            simplifiedMainMenu ? { width: '45px', borderRadius: '0 10px 10px 0' } : mainMenuWidth,
          ]"
        >
          <v-window v-model="interfaceStore.mainMenuCurrentStep" class="h-full w-full">
            <v-window-item :value="1" class="h-full">
              <div
                class="relative flex flex-col h-full justify-between align-center items-center select-none"
                :class="
                  interfaceStore.isOnSmallScreen
                    ? 'gap-y-0 pt-2 pb-3 sm:sm:py-0 sm:-ml-[3px] xs:xs:py-0 xs:-ml-[3px]'
                    : 'lg:gap-y-2 xl:gap-y-3 gap-y-4 py-4'
                "
              >
                <GlassButton
                  v-if="route.name === 'widgets-view'"
                  :label="simplifiedMainMenu ? '' : 'Edit Interface'"
                  :selected="widgetStore.editingMode"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? 'mdi-pencil' : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? 'Edit Mode' : undefined"
                  :width="buttonSize"
                  @click="
                    () => {
                      widgetStore.editingMode = !widgetStore.editingMode
                      closeMainMenu()
                    }
                  "
                  ><img v-if="!simplifiedMainMenu" :src="EditModeIcon" alt="Edit Mode Icon" />
                </GlassButton>
                <GlassButton
                  v-if="route.name !== 'widgets-view'"
                  :label="simplifiedMainMenu ? '' : 'Flight'"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? 'mdi-send' : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? 'Flight' : undefined"
                  :width="buttonSize"
                  :selected="$route.name === 'Flight'"
                  @click="
                    () => {
                      $router.push('/')
                      closeMainMenu()
                    }
                  "
                  ><img v-if="!simplifiedMainMenu" :src="FlightIcon" alt="Flight Icon" />
                </GlassButton>
                <GlassButton
                  v-if="route.name !== 'Mission planning'"
                  :label="simplifiedMainMenu ? '' : 'Mission Planning'"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? 'mdi-map-marker-radius-outline' : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? 'Mission Planning' : undefined"
                  :width="buttonSize"
                  :selected="$route.name === 'Mission planning'"
                  @click="
                    () => {
                      $router.push('/mission-planning')
                      closeMainMenu()
                    }
                  "
                  ><img v-if="!simplifiedMainMenu" :src="MissionPlanningIcon" alt="MissionPlanning Icon"
                /></GlassButton>
                <GlassButton
                  :label="simplifiedMainMenu ? '' : 'Settings'"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? 'mdi-cog' : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? 'Configuration' : undefined"
                  :width="buttonSize"
                  :selected="showSubMenu"
                  class="mb-1"
                  :style="
                    interfaceStore.highlightedComponent === 'settings-menu-item' && {
                      animation: 'highlightBackground 0.5s alternate 20',
                      borderRadius: '10px',
                    }
                  "
                  @click="selectSubMenu(SubMenuName.settings)"
                  ><img v-if="!simplifiedMainMenu" :src="SettingsIcon" alt="Settings Icon" />
                </GlassButton>
                <GlassButton
                  :label="simplifiedMainMenu ? '' : 'Tools'"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? 'mdi-tools' : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? 'Tools' : undefined"
                  :width="buttonSize"
                  :selected="showSubMenu"
                  class="mb-1"
                  @click="selectSubMenu(SubMenuName.tools)"
                  ><img v-if="!simplifiedMainMenu" :src="ToolsIcon" alt="Tools Icon" />
                </GlassButton>
                <GlassButton
                  :label="simplifiedMainMenu ? '' : isFullscreen ? 'Exit Fullscreen' : 'Enter Fullscreen'"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? fullScreenToggleIcon : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? (isFullscreen ? 'Exit Fullscreen' : 'Enter Fullscreen') : undefined"
                  :button-class="simplifiedMainMenu ? '-mb-2' : ''"
                  :width="buttonSize"
                  :selected="false"
                  @click="
                    () => {
                      toggleFullscreen()
                      closeMainMenu()
                    }
                  "
                  ><img
                    v-if="!simplifiedMainMenu"
                    :src="isFullscreen ? ExitFullScreenIcon : FullScreenIcon"
                    alt="Fullscreen Icon"
                  />
                </GlassButton>
                <GlassButton
                  :label="simplifiedMainMenu ? '' : 'About'"
                  :label-class="[menuLabelSize, '-mb-0.5 mt-6']"
                  :icon="simplifiedMainMenu ? 'mdi-information-outline' : undefined"
                  :icon-size="simplifiedMainMenu ? 25 : undefined"
                  variant="uncontained"
                  :tooltip="simplifiedMainMenu ? 'About' : undefined"
                  :button-class="!simplifiedMainMenu ? '-mt-[5px]' : undefined"
                  :width="buttonSize"
                  :selected="showSubMenu"
                  @click="openAboutDialog"
                  ><img v-if="!simplifiedMainMenu" :src="InfoIcon" alt="Info Icon" />
                </GlassButton>
              </div>
            </v-window-item>
            <v-window-item :value="2" class="h-full w-full">
              <div class="flex flex-col w-full h-full justify-between">
                <GlassButton
                  v-for="menuitem in currentSubMenu"
                  :key="menuitem.title"
                  :label="simplifiedMainMenu ? undefined : menuitem.title"
                  :label-class="menuLabelSize"
                  :button-class="interfaceStore.isOnSmallScreen ? '-ml-[2px]' : ''"
                  :icon="menuitem.icon"
                  :selected="interfaceStore.currentSubMenuComponentName === menuitem.componentName"
                  variant="uncontained"
                  :height="buttonSize * 0.45"
                  :icon-size="buttonSize * 0.5"
                  :style="
                    interfaceStore.highlightedComponent === menuitem.title && {
                      animation: 'highlightBackground 0.5s alternate 50',
                      borderRadius: '4px',
                    }
                  "
                  @click="toggleSubMenuComponent(menuitem.component)"
                  ><template #content
                    ><div v-if="currentSubMenuComponent === menuitem.component" class="arrow-left"></div></template
                ></GlassButton>
                <div class="flex flex-col justify-center align-center pb-1">
                  <v-divider width="70%" />
                  <GlassButton
                    :label-class="menuLabelSize"
                    icon="mdi-arrow-left"
                    :icon-class="interfaceStore.isOnSmallScreen ? '' : '-mb-[1px]'"
                    :button-class="interfaceStore.isOnSmallScreen ? (simplifiedMainMenu ? '-mt-1' : 'mt-1') : undefined"
                    variant="round"
                    :width="buttonSize / 2.4"
                    :selected="false"
                    @click="
                      () => {
                        interfaceStore.mainMenuCurrentStep = 1
                        currentSubMenuComponent = null
                      }
                    "
                  />
                </div>
              </div>
            </v-window-item>
          </v-window>
        </div>
      </transition>

      <teleport to="body">
        <GlassModal
          :is-visible="
            currentSubMenuComponent !== null &&
            interfaceStore.mainMenuCurrentStep === 2 &&
            interfaceStore.isMainMenuVisible
          "
          position="menuitem"
          :class="interfaceStore.isVideoLibraryVisible ? 'opacity-0' : 'opacity-100'"
          @close-modal="currentSubMenuComponent = null"
        >
          <component :is="currentSubMenuComponent"></component>
        </GlassModal>
      </teleport>

      <div ref="routerSection" class="router-view">
        <div class="main-view" :class="{ 'edit-mode': widgetStore.editingMode }" :style="connectionStatusFeedback">
          <div
            v-show="showTopBarNow"
            id="mainTopBar"
            class="bar top-bar"
            :style="[
              interfaceStore.globalGlassMenuStyles,
              interfaceStore.isOnSmallScreen && interfaceStore.isOnSmallScreen ? topBarScaleStyle : undefined,
            ]"
          >
            <button
              v-if="interfaceStore.mainMenuStyleTrigger === 'burger'"
              class="flex items-center justify-center h-full mr-2 aspect-square top-bar-hamburger"
              @click="toggleMainMenu"
            >
              <span class="text-3xl transition-all mdi mdi-menu text-slate-300 hover:text-slate-50" />
            </button>
            <div class="flex-1">
              <MiniWidgetContainer
                :container="widgetStore.currentMiniWidgetsProfile.containers[0]"
                :allow-editing="widgetStore.editingMode"
                align="start"
              />
            </div>
            <div class="grow" />
            <div class="flex-1">
              <MiniWidgetContainer
                :container="widgetStore.currentMiniWidgetsProfile.containers[1]"
                :allow-editing="widgetStore.editingMode"
                align="center"
              />
            </div>
            <div class="grow" />
            <div class="flex-1">
              <MiniWidgetContainer
                :container="widgetStore.currentMiniWidgetsProfile.containers[2]"
                :allow-editing="widgetStore.editingMode"
                align="end"
              />
            </div>
          </div>
          <AltitudeSlider />
          <div class="bottom-container">
            <SlideToConfirm />
          </div>
          <div v-for="view in widgetStore.viewsToShow" :key="view.name">
            <Transition name="fade">
              <div
                v-show="view.name === currentSelectedViewName && showBottomBarNow"
                class="bar bottom-bar"
                :style="[
                  interfaceStore.globalGlassMenuStyles,
                  interfaceStore.isOnSmallScreen ? bottomBarScaleStyle : undefined,
                ]"
              >
                <MiniWidgetContainer
                  :container="view.miniWidgetContainers[0]"
                  :allow-editing="widgetStore.editingMode"
                  align="start"
                />
                <div />
                <MiniWidgetContainer
                  :container="view.miniWidgetContainers[1]"
                  :allow-editing="widgetStore.editingMode"
                  align="center"
                />
                <div />
                <MiniWidgetContainer
                  :container="view.miniWidgetContainers[2]"
                  :allow-editing="widgetStore.editingMode"
                  align="end"
                />
              </div>
            </Transition>
          </div>
          <router-view />
        </div>
        <EditMenu v-model:edit-mode="widgetStore.editingMode" />
      </div>
    </v-main>
  </v-app>
  <About v-if="showAboutDialog" @update:show-about-dialog="showAboutDialog = $event" />
  <Tutorial v-if="interfaceStore.isTutorialVisible" />
  <VideoLibraryModal v-if="interfaceStore.isVideoLibraryVisible" />
  <VehicleDiscoveryDialog v-model="showDiscoveryDialog" show-auto-search-option />
  <UpdateNotification v-if="isElectron()" />
  <SnackbarContainer />
</template>

<script setup lang="ts">
import { onClickOutside, useFullscreen, useStorage, useWindowSize } from '@vueuse/core'
import { computed, markRaw, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { useRoute } from 'vue-router'

import EditModeIcon from '@/assets/icons/edit-mode.svg'
import ExitFullScreenIcon from '@/assets/icons/exit-full-screen.svg'
import FlightIcon from '@/assets/icons/flight.svg'
import FullScreenIcon from '@/assets/icons/full-screen.svg'
import InfoIcon from '@/assets/icons/info.svg'
import MissionPlanningIcon from '@/assets/icons/mission-planning.svg'
import SettingsIcon from '@/assets/icons/settings.svg'
import ToolsIcon from '@/assets/icons/tools.svg'
import GlassModal from '@/components/GlassModal.vue'
import SnackbarContainer from '@/components/SnackbarContainer.vue'
import Tutorial from '@/components/Tutorial.vue'
import UpdateNotification from '@/components/UpdateNotification.vue'
import VehicleDiscoveryDialog from '@/components/VehicleDiscoveryDialog.vue'
import VideoLibraryModal from '@/components/VideoLibraryModal.vue'
import { useInteractionDialog } from '@/composables/interactionDialog'
import {
  availableCockpitActions,
  registerActionCallback,
  unregisterActionCallback,
} from '@/libs/joystick/protocols/cockpit-actions'
import { isElectron } from '@/libs/utils'

import About from './components/About.vue'
import AltitudeSlider from './components/AltitudeSlider.vue'
import EditMenu from './components/EditMenu.vue'
import GlassButton from './components/GlassButton.vue'
import MiniWidgetContainer from './components/MiniWidgetContainer.vue'
import SlideToConfirm from './components/SlideToConfirm.vue'
import { useSnackbar } from './composables/snackbar'
import { SubMenuComponentName, SubMenuName, useAppInterfaceStore } from './stores/appInterface'
import { useMainVehicleStore } from './stores/mainVehicle'
import { useWidgetManagerStore } from './stores/widgetManager'
import { SubMenuComponent } from './types/general'
import ConfigurationActionsView from './views/ConfigurationActionsView.vue'
import ConfigurationAlertsView from './views/ConfigurationAlertsView.vue'
import ConfigurationDevelopmentView from './views/ConfigurationDevelopmentView.vue'
import ConfigurationGeneralView from './views/ConfigurationGeneralView.vue'
import ConfigurationJoystickView from './views/ConfigurationJoystickView.vue'
import ConfigurationTelemetryView from './views/ConfigurationLogsView.vue'
import ConfigurationMissionView from './views/ConfigurationMissionView.vue'
import ConfigurationUIView from './views/ConfigurationUIView.vue'
import ConfigurationVideoView from './views/ConfigurationVideoView.vue'
import ToolsDataLakeView from './views/ToolsDataLakeView.vue'
import ToolsMAVLinkView from './views/ToolsMAVLinkView.vue'
const { showDialog, closeDialog } = useInteractionDialog()
const { openSnackbar } = useSnackbar()

const widgetStore = useWidgetManagerStore()
const vehicleStore = useMainVehicleStore()
const interfaceStore = useAppInterfaceStore()

const showAboutDialog = ref(false)
const showSubMenu = ref(false)
const currentSubMenuComponent = ref<SubMenuComponent>(null)
const mainMenu = ref()

// Main menu
const isMenuOpen = ref(false)
const isSlidingOut = ref(false)

const { width: windowWidth, height: windowHeight } = useWindowSize()

const configMenu = [
  {
    icon: 'mdi-view-dashboard-variant',
    title: 'General',
    componentName: SubMenuComponentName.SettingsGeneral,
    component: markRaw(ConfigurationGeneralView) as SubMenuComponent,
  },
  {
    icon: 'mdi-monitor-cellphone',
    title: 'Interface',
    componentName: SubMenuComponentName.SettingsInterface,
    component: markRaw(ConfigurationUIView) as SubMenuComponent,
  },
  {
    icon: 'mdi-controller',
    title: 'Joystick',
    componentName: SubMenuComponentName.SettingsJoystick,
    component: markRaw(ConfigurationJoystickView) as SubMenuComponent,
  },
  {
    icon: 'mdi-video',
    title: 'Video',
    componentName: SubMenuComponentName.SettingsVideo,
    component: markRaw(ConfigurationVideoView) as SubMenuComponent,
  },
  {
    icon: 'mdi-subtitles-outline',
    title: 'Telemetry',
    componentName: SubMenuComponentName.SettingsTelemetry,
    component: markRaw(ConfigurationTelemetryView) as SubMenuComponent,
  },
  {
    icon: 'mdi-alert-rhombus-outline',
    title: 'Alerts',
    componentName: SubMenuComponentName.SettingsAlerts,
    component: markRaw(ConfigurationAlertsView) as SubMenuComponent,
  },
  {
    icon: 'mdi-dev-to',
    title: 'Dev',
    componentName: SubMenuComponentName.SettingsDev,
    component: markRaw(ConfigurationDevelopmentView) as SubMenuComponent,
  },
  {
    icon: 'mdi-map-marker-path',
    title: 'Mission',
    componentName: SubMenuComponentName.SettingsMission,
    component: markRaw(ConfigurationMissionView) as SubMenuComponent,
  },
  {
    icon: 'mdi-run-fast',
    title: 'Actions',
    componentName: SubMenuComponentName.SettingsActions,
    component: markRaw(ConfigurationActionsView) as SubMenuComponent,
  },
]

const toolsMenu = [
  {
    icon: 'mdi-protocol',
    title: 'MAVLink',
    componentName: SubMenuComponentName.ToolsMAVLink,
    component: markRaw(ToolsMAVLinkView) as SubMenuComponent,
  },
  {
    icon: 'mdi-database-outline',
    title: 'Data-lake',
    componentName: SubMenuComponentName.ToolsDataLake,
    component: markRaw(ToolsDataLakeView) as SubMenuComponent,
  },
]

const availableSubMenus = {
  settings: configMenu,
  tools: toolsMenu,
}

const currentSubMenu = computed(() => {
  if (interfaceStore.currentSubMenuName === null) return []
  return availableSubMenus[interfaceStore.currentSubMenuName]
})

watch(
  () => interfaceStore.currentSubMenuComponentName,
  (subMenuComponentName) => {
    currentSubMenuComponent.value =
      currentSubMenu.value.find((item) => item.componentName === subMenuComponentName)?.component || null
  }
)

const selectSubMenu = (subMenuName: SubMenuName): void => {
  interfaceStore.currentSubMenuName = subMenuName
  interfaceStore.mainMenuCurrentStep = 2
}

const toggleSubMenuComponent = (component: SubMenuComponent): void => {
  if (currentSubMenuComponent.value === null) {
    currentSubMenuComponent.value = component
    interfaceStore.configModalVisibility = true
    return
  }
  if (currentSubMenuComponent.value === component) {
    currentSubMenuComponent.value = null
    interfaceStore.configModalVisibility = false
    return
  }
  currentSubMenuComponent.value = component
  interfaceStore.configModalVisibility = true
}

const isConfigModalVisible = computed(() => interfaceStore.isConfigModalVisible)

watch(isConfigModalVisible, (newVal) => {
  if (newVal === false) {
    currentSubMenuComponent.value = null
  }
})

const topBottomBarScale = computed(() => {
  return windowWidth.value / originalBarWidth
})

const maxScreenHeightPixelsThatFitsLargeMenu = computed(() => {
  const heightTopBar = widgetStore.currentTopBarHeightPixels * topBottomBarScale.value
  const heightBottomBar = widgetStore.currentBottomBarHeightPixels * topBottomBarScale.value
  const visibleAreaHeight = windowHeight.value - heightTopBar - heightBottomBar
  return visibleAreaHeight
})

const simplifiedMainMenu = computed(() => {
  const threshold = windowWidth.value > 1300 ? 860 : 680
  return maxScreenHeightPixelsThatFitsLargeMenu.value < threshold
})

const mainMenuWidth = computed(() => {
  const width =
    interfaceStore.isOnSmallScreen && interfaceStore.mainMenuCurrentStep === 2
      ? '60px'
      : `${interfaceStore.mainMenuWidth}px`
  return { width }
})

const toggleMainMenu = (): void => {
  if (isMenuOpen.value === true) {
    closeMainMenu()
  } else {
    openMainMenu()
  }
}

const openMainMenu = (): void => {
  if (vehicleStore.isArmed) {
    showDialog({
      title: 'Be careful',
      maxWidth: '650px',
      message: 'The vehicle is currently armed and it is not recommended to open the main menu.',
      actions: [
        {
          text: 'Continue anyway',
          action: () => {
            interfaceStore.isMainMenuVisible = true
            isMenuOpen.value = true
            closeDialog()
          },
        },
        {
          text: 'Disarm vehicle',
          action: () => {
            disarmVehicle()
          },
        },
      ],
      variant: 'warning',
    }).then((result) => {
      if (result.isConfirmed && vehicleStore.isArmed) {
        vehicleStore.disarm().then(() => {
          interfaceStore.isMainMenuVisible = true
          isMenuOpen.value = true
        })
      } else {
        interfaceStore.isMainMenuVisible = true
        isMenuOpen.value = true
      }
    })
  } else {
    interfaceStore.isMainMenuVisible = true
    isMenuOpen.value = true
  }
}

// Close Main Menu Logic
const closeMainMenu = (): void => {
  isSlidingOut.value = true
  setTimeout(() => {
    interfaceStore.isMainMenuVisible = false
    isSlidingOut.value = false
    isMenuOpen.value = false
    interfaceStore.mainMenuCurrentStep = 1
    currentSubMenuComponent.value = null
  }, 20)
}

const disarmVehicle = (): void => {
  vehicleStore.disarm().then(() => {
    interfaceStore.isMainMenuVisible = true
  })
  closeDialog()
}

const handleEscKey = (event: KeyboardEvent): void => {
  if (event.key === 'Escape' && interfaceStore.isMainMenuVisible) {
    closeMainMenu()
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleEscKey)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', handleEscKey)
})

/* eslint-disable jsdoc/require-jsdoc  */
const connectionStatusFeedback = ref<{ border: string; transition?: string }>({ border: '0px' })

const resetConnectionStatusFeedback = (): void => {
  setTimeout(() => {
    connectionStatusFeedback.value = {
      border: '0px solid transparent',
      transition: 'border 4s ease-out',
    }
  }, 4000)
}

// Connection monitoring and visual feedback
watch(
  () => vehicleStore.isVehicleOnline,
  (isOnline) => {
    if (!isOnline) {
      openSnackbar({
        message: 'Vehicle connection lost: reestablishing',
        variant: 'error',
        duration: 3000,
        closeButton: false,
      })
      connectionStatusFeedback.value = { border: '3px solid red' }

      resetConnectionStatusFeedback()
      return
    }

    openSnackbar({ message: 'Vehicle connected', variant: 'success', duration: 3000, closeButton: false })
    connectionStatusFeedback.value = { border: '3px solid green' }

    resetConnectionStatusFeedback()
  }
)

const buttonSize = computed(() => {
  if (interfaceStore.is2xl) return 72
  if (interfaceStore.isXl) return 66
  if (interfaceStore.isLg) return 60
  if (interfaceStore.isMd) return 54
  if (interfaceStore.isSm && windowHeight.value > 700) return 60
  if (interfaceStore.isSm && windowHeight.value < 700) return 48
  if (interfaceStore.isXs && windowHeight.value >= 700) return 60
  return 48
})

const menuLabelSize = computed(() => {
  if (interfaceStore.is2xl) return 'text-[15px]'
  if (interfaceStore.isXl) return 'text-[14px]'
  if (interfaceStore.isLg) return 'text-[13px]'
  if (interfaceStore.isMd) return 'text-[12px]'
  if (interfaceStore.isSm) return 'text-[10px]'
  if (interfaceStore.isXs && windowHeight.value >= 700) return 'text-[12px]'
  return 'text-[10px]'
})

onClickOutside(mainMenu, () => {
  if (interfaceStore.mainMenuCurrentStep === 1 && !interfaceStore.isTutorialVisible) {
    closeMainMenu()
  }
  if (
    interfaceStore.mainMenuCurrentStep === 2 &&
    currentSubMenuComponent.value === null &&
    !interfaceStore.isTutorialVisible
  ) {
    closeMainMenu()
  }
})

const glassMenuStyles = computed(() => ({
  backgroundColor: interfaceStore.UIGlassEffect.bgColor,
  color: interfaceStore.UIGlassEffect.fontColor,
  backdropFilter: `blur(${interfaceStore.UIGlassEffect.blur}px)`,
}))

const openAboutDialog = (): void => {
  showAboutDialog.value = true
  closeMainMenu()
}

const route = useRoute()
const routerSection = ref()

// Full screen toggling
const { isFullscreen, toggle: toggleFullscreen } = useFullscreen()

const fullScreenCallbackId = registerActionCallback(availableCockpitActions.toggle_full_screen, toggleFullscreen)
onBeforeUnmount(() => unregisterActionCallback(fullScreenCallbackId))

const fullScreenToggleIcon = computed(() => (isFullscreen.value ? 'mdi-fullscreen-exit' : 'mdi-overscan'))

const currentSelectedViewName = computed(() => widgetStore.currentView.name)

const originalBarWidth = 1800

const topBarScaleStyle = computed(() => {
  return {
    transform: `scale(${topBottomBarScale.value})`,
    transformOrigin: 'top left',
    width: `${originalBarWidth}px`,
  }
})

const bottomBarScaleStyle = computed(() => {
  return {
    transform: `scale(${topBottomBarScale.value})`,
    transformOrigin: 'bottom left',
    width: `${originalBarWidth}px`,
  }
})

// Control showing mouse
let hideMouseTimeoutId: ReturnType<typeof setInterval>

const hideMouse = (): void => {
  document.body.classList.add('hide-cursor')
}

const resetHideMouseTimeout = (): void => {
  clearTimeout(hideMouseTimeoutId)
  document.body.classList.remove('hide-cursor')
  hideMouseTimeoutId = setTimeout(hideMouse, 5000)
}

document.addEventListener('mousemove', resetHideMouseTimeout)

// Control top/bottom bar momentary hiding
const showBottomBarNow = ref(widgetStore.currentView.showBottomBarOnBoot)
const showTopBarNow = ref(true)
watch([() => widgetStore.currentView, () => widgetStore.currentView.showBottomBarOnBoot], () => {
  showBottomBarNow.value = widgetStore.currentView.showBottomBarOnBoot
})
const bottomBarToggleCallbackId = registerActionCallback(
  availableCockpitActions.toggle_bottom_bar,
  () => (showBottomBarNow.value = !showBottomBarNow.value)
)
const topBarToggleCallbackId = registerActionCallback(
  availableCockpitActions.toggle_top_bar,
  () => (showTopBarNow.value = !showTopBarNow.value)
)
onBeforeUnmount(() => {
  unregisterActionCallback(bottomBarToggleCallbackId)
  unregisterActionCallback(topBarToggleCallbackId)
})

// Dynamic styles
const currentTopBarHeightPixels = computed(() => `${widgetStore.currentTopBarHeightPixels}px`)
const currentBottomBarHeightPixels = computed(() => `${widgetStore.currentBottomBarHeightPixels}px`)

const showDiscoveryDialog = ref(false)
const preventAutoSearch = useStorage('cockpit-prevent-auto-vehicle-discovery-dialog', false)

onMounted(() => {
  if (isElectron() && !preventAutoSearch.value) {
    // Wait 5 seconds to check if we're connected to a vehicle
    setTimeout(() => {
      if (vehicleStore.isVehicleOnline) return
      showDiscoveryDialog.value = true
    }, 5000)
  }

  if (!interfaceStore.userHasSeenTutorial) {
    setTimeout(() => {
      interfaceStore.isTutorialVisible = true
    }, 6000)
  }
})
</script>

<style>
html,
body {
  /* Removes the scrollbar */
  overflow: hidden !important;
}

body.hide-cursor {
  cursor: none;
}

.left-menu {
  position: fixed;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-radius: 0 20px 20px 0;
  border: 1px #cbcbcb22 solid;
  border-left: none;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  box-shadow: 0px 4px 4px 0px rgba(0, 0, 0, 0.3), 0px 8px 12px 6px rgba(0, 0, 0, 0.15);
  z-index: 1000;
}

@keyframes slideInLeft {
  from {
    transform: translateX(-100%) translateY(-50%);
  }
  to {
    transform: translateX(0) translateY(-50%);
  }
}

@keyframes slideOutLeft {
  from {
    transform: translateX(0) translateY(-50%);
  }
  to {
    transform: translateX(-100%) translateY(-50%);
  }
}

.slide-in-left-enter-active {
  animation: slideInLeft 300ms ease forwards;
}

.slide-in-left-leave-active {
  animation: slideOutLeft 300ms ease forwards;
}

.router-view {
  width: 100%;
  height: 100%;
  position: relative;
}

.main-view {
  transition: all 0.2s;
  width: 100%;
  height: 100%;
  position: absolute;
  right: 0%;
  top: 0%;
}

.main-view.edit-mode {
  transform: scale(0.78);
  right: -11%;
  top: -11%;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s cubic-bezier(0.55, 0, 0.1, 1);
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translate(0, 100px);
}

.bottom-container {
  position: absolute;
  bottom: v-bind('currentBottomBarHeightPixels');
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 60;
}

.bar {
  width: 100%;
  display: flex;
  justify-content: space-between;
  z-index: 60;
  position: absolute;
}

.menu-trigger {
  position: fixed;
  left: 0;
  transform: translateY(-50%);
  z-index: 1050;
}

.bottom-bar {
  bottom: 0;
  height: v-bind('currentBottomBarHeightPixels');
}

.top-bar {
  top: 0;
  height: v-bind('currentTopBarHeightPixels');
}

.top-bar-hamburger {
  outline: none;
}
</style>
