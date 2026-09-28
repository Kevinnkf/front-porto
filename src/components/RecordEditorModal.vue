<template>
  <Teleport to="body">
    <div
      v-if="open"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm p-4"
      @click.self="$emit('cancel')"
    >
      <form
        class="w-full max-w-lg max-h-[90vh] overflow-y-auto rounded-2xl bg-white dark:bg-gray-800 p-6 shadow-2xl"
        @submit.prevent="$emit('save', { ...form })"
      >
        <h2 class="mb-5 text-2xl font-bold text-gray-900 dark:text-white">{{ title }}</h2>
        <div v-for="field in fields" :key="field.key" class="mb-4 flex flex-col gap-1">
          <label :for="`record-${field.key}`" class="text-sm font-medium text-gray-700 dark:text-gray-300">
            {{ field.label }}
          </label>
          <textarea
            v-if="field.type === 'textarea'"
            :id="`record-${field.key}`"
            v-model="form[field.key]"
            :required="field.required !== false"
            rows="4"
            class="w-full rounded-lg border border-gray-300 bg-white px-3 py-2 text-gray-900 dark:border-gray-600 dark:bg-gray-700 dark:text-white"
          ></textarea>
          <input
            v-else
            :id="`record-${field.key}`"
            v-model="form[field.key]"
            :type="field.type || 'text'"
            :required="field.required !== false"
            class="w-full rounded-lg border border-gray-300 bg-white px-3 py-2 text-gray-900 dark:border-gray-600 dark:bg-gray-700 dark:text-white"
          />
        </div>
        <p v-if="error" class="mb-4 rounded-lg bg-red-50 px-3 py-2 text-sm text-red-700 dark:bg-red-900/30 dark:text-red-300">
          {{ error }}
        </p>
        <div class="flex justify-end gap-3">
          <button
            type="button"
            class="rounded-lg border border-gray-300 px-4 py-2 text-sm text-gray-700 dark:border-gray-600 dark:text-gray-200"
            :disabled="saving"
            @click="$emit('cancel')"
          >Cancel</button>
          <button
            type="submit"
            class="rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 disabled:opacity-60"
            :disabled="saving"
          >{{ saving ? 'Saving…' : 'Save' }}</button>
        </div>
      </form>
    </div>
  </Teleport>
</template>

<script>
import { reactive, watch } from 'vue'

export default {
  name: 'RecordEditorModal',
  props: {
    open: Boolean,
    title: { type: String, required: true },
    fields: { type: Array, required: true },
    initialValue: { type: Object, default: () => ({}) },
    saving: Boolean,
    error: { type: String, default: '' }
  },
  emits: ['save', 'cancel'],
  setup(props) {
    const form = reactive({})

    watch(() => props.open, (isOpen) => {
      if (!isOpen) return
      for (const key of Object.keys(form)) delete form[key]
      Object.assign(form, props.initialValue || {})
    })

    return { form }
  }
}
</script>
