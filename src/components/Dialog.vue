<script setup lang="ts">
import { Icon } from '@iconify/vue'
import { onKeyStroke } from '@vueuse/core'

const props = withDefaults(defineProps<{
  title: string
  desc?: string
  center?: boolean
  appendToBewlyBody?: boolean
  width?: string | number
  height?: string | number
  centerFooter?: boolean
  loading?: boolean
  preventCloseWhenLoading?: boolean
}>(), {
  preventCloseWhenLoading: true,
})

const emit = defineEmits(['close', 'confirm'])

onKeyStroke('Enter', (e: KeyboardEvent) => {
  e.preventDefault()
  if (!props.loading)
    handleConfirm()
})
onKeyStroke('Escape', (e: KeyboardEvent) => {
  e.preventDefault()
  if (props.loading && props.preventCloseWhenLoading)
    return
  handleClose()
})

const showShortcut = ref<boolean>(false)
const { mainAppRef } = useBewlyApp()
const showDialog = ref<boolean>(false)

const dialogWidth = computed(() => {
  return typeof props.width === 'number' ? `${props.width}px` : props.width || '400px'
})
const dialogHeight = computed(() => {
  return typeof props.height === 'number' ? `${props.height}px` : props.height || 'auto'
})

onKeyStroke('Alt', (e: KeyboardEvent) => {
  e.preventDefault()
  showShortcut.value = true
}, { eventName: 'keydown' })
onKeyStroke('Alt', (e: KeyboardEvent) => {
  e.preventDefault()
  showShortcut.value = false
}, { eventName: 'keyup' })

onMounted(() => {
  showDialog.value = true
})

onBeforeUnmount(() => {
  handleClose()
})

function handleClose() {
  if (props.loading && props.preventCloseWhenLoading)
    return

  showDialog.value = false
  nextTick(() => {
    emit('close')
  })
}

function handleConfirm() {
  emit('confirm')
  if (!props.loading)
    handleClose()
}
</script>

<template>
  <Teleport :to="mainAppRef" :disabled="!appendToBewlyBody">
    <Transition name="modal">
      <div
        v-if="showDialog"
        class="dialog"
        pos="fixed top-0 left-0" w-full h-full z-100
      >
        <div
          bg="black opacity-40 dark:opacity-40"
          pos="absolute top-0 left-0" w-full h-full z-0
          @click="handleClose"
        />
        <div
          style="
            box-shadow: var(--bew-shadow-3) var(--bew-shadow-edge-glow-2);
            backdrop-filter: var(--bew-filter-glass-2);
          "
          :style="{ width: dialogWidth, height: dialogHeight }"
          pos="absolute top-1/2 left-1/2" bg="$bew-elevated-1" rounded="$bew-radius"
          transform="translate--1/2" z-2 overflow="x-hidden y-overlay"
          antialiased
        >
          <!-- loading masking -->
          <Transition name="fade">
            <div
              v-if="loading"
              pos="absolute top-0 left-0" w-full h-full bg="white dark:black opacity-60 dark:opacity-60" flex="~ justify-center items-center"
              z-2
            >
              <Icon icon="svg-spinners:ring-resize" text="4xl" />
            </div>
          </Transition>

          <header
            style="
              text-shadow: 0 0 15px var(--bew-elevated-solid-1), 0 0 20px var(--bew-elevated-solid-1)
            "
            pos="sticky top-0 left-0" w-full h-70px px-8 flex
            items-center justify-between
            rounded="t-$bew-radius" z-1
          >
            <div
              :style="{ textAlign: center ? 'center' : 'left' }"
              w-full
            >
              <p text-xl fw-bold>
                {{ title }}
              </p>
              <p text="sm $bew-text-2">
                <slot name="desc">
                  {{ desc }}
                </slot>
              </p>
            </div>

            <div
              style="backdrop-filter: var(--bew-filter-glass-1)"
              text-2xl leading-0 bg="$bew-fill-1 hover:$bew-theme-color-30" w="32px" h="32px"
              ml-8 p="1" rounded-8 cursor="pointer"
              hover:ring="2 $bew-theme-color" hover:text="$bew-theme-color" duration-300
              @click="handleClose"
            >
              <ic-baseline-clear />
            </div>
          </header>
          <main p="x-8 y-2" relative>
            <!-- <div h-80px mt--8 /> -->
            <slot />
          </main>
          <footer
            :style="{ justifyContent: centerFooter || center ? 'center' : 'flex-end' }"
            flex="~ gap-2" p="x-8 t-6 b-6"
          >
            <Button type="tertiary" @click="handleClose">
              <div>
                {{ $t('common.cancel') }}
                <span v-show="showShortcut" text="xs $bew-text-2">(Esc)</span>
              </div>
            </Button>
            <Button type="primary" @click="handleConfirm">
              <div>
                {{ $t('common.confirm') }}
                <span v-show="showShortcut" text="xs">(Enter)</span>
              </div>
            </Button>
          </footer>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style lang="scss" scoped>
</style>
