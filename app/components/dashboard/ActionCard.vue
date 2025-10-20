<script setup lang="ts">
interface Props {
  title: string
  linkedTo: string // Nom de l'entité, audit ou OE
  linkedToType: 'entity' | 'audit' | 'oe' // Type de lien
  dateLimite: string // Date limite au format YYYY-MM-DD
  color?: string
  bgColor?: string
}

const props = withDefaults(defineProps<Props>(), {
  color: 'border-primary-500',
  bgColor: 'bg-primary-50'
})

// Formater la date limite
const formatDateLimite = (date: string) => {
  const d = new Date(date)
  return d.toLocaleDateString('fr-FR', { day: '2-digit', month: '2-digit', year: 'numeric' })
}
</script>

<template>
  <div
    class="rounded-lg border-l-4 shadow-sm p-3 transition-all hover:shadow-md"
    :class="[color, bgColor]"
  >
    <div class="flex flex-col gap-1.5">
      <div class="font-semibold text-sm text-gray-800">
        {{ title }}
      </div>
      <div class="text-xs text-gray-600 flex items-center gap-1">
        <span class="font-medium">
          {{ linkedToType === 'entity' ? 'Entité' : linkedToType === 'audit' ? 'Audit' : 'OE' }}:
        </span>
        <span>{{ linkedTo }}</span>
      </div>
      <div class="text-xs font-medium text-gray-700 flex items-center gap-1">
        <span class="opacity-70">Date limite:</span>
        <span>{{ formatDateLimite(dateLimite) }}</span>
      </div>
    </div>
  </div>
</template>
