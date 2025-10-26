<script setup lang="ts">
import { computed } from 'vue'

type Variant = 'primary' | 'secondary' | 'success' | 'destructive' | 'outline' | 'ghost'
type Size = 'sm' | 'md' | 'lg' | 'icon'

const props = withDefaults(defineProps<{
  variant?: Variant
  size?: Size
  loading?: boolean
  disabled?: boolean
  block?: boolean
}>(), {
  variant: 'primary',
  size: 'md',
  loading: false,
  disabled: false,
  block: false
})

const base =
  'inline-flex items-center justify-center gap-2 rounded-xl font-semibold transition ' +
  'focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 ' +
  'disabled:opacity-60 disabled:cursor-not-allowed active:scale-[.98] ' +
  'ring-offset-white'

const variants: Record<Variant, string> = {
  primary:    'bg-indigo-600 text-white hover:bg-indigo-700 focus-visible:ring-indigo-600 shadow-sm',
  secondary:  'bg-gray-900 text-white hover:bg-black focus-visible:ring-gray-900 shadow-sm',
  success:    'bg-teal-600 text-white hover:bg-teal-700 focus-visible:ring-teal-600 shadow-sm',
  destructive:'bg-rose-600 text-white hover:bg-rose-700 focus-visible:ring-rose-600 shadow-sm',
  outline:    'border border-gray-300 text-gray-900 hover:bg-gray-50 focus-visible:ring-gray-900',
  ghost:      'text-gray-700 hover:bg-gray-100 focus-visible:ring-gray-300'
}

const sizes: Record<Size, string> = {
  sm: 'text-sm px-3 py-1.5',
  md: 'text-sm px-3.5 py-2.5',
  lg: 'text-base px-4.5 py-3',
  icon: 'p-2'
}

const cls = computed(() => [
  base,
  variants[props.variant],
  sizes[props.size],
  props.block ? 'w-full' : ''
].join(' '))
</script>

<template>
  <button :class="cls" :disabled="disabled || loading">
    <svg
      v-if="loading"
      class="-ms-1 me-1.5 h-4 w-4 animate-spin"
      viewBox="0 0 24 24" fill="none" aria-hidden="true"
    >
      <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
      <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 0 1 8-8v4a4 4 0 0 0-4 4H4z"/>
    </svg>
    <slot/>
  </button>
</template>
