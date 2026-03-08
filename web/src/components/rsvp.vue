<script setup lang="ts">
import { ref, onMounted } from "vue"

type Row = {
  ID: string
  Name: string
  "RSVP status": string
  Allergies: string
  Notes: string
}

type ApiResponse =
  | { ok: true; row: Row }
  | { ok: false; error: string }

const API = "https://script.google.com/macros/s/AKfycbwVDlXkhmvtzFhoIJvNaBqqXTIQQQ8pPhwtn3vx0nkePPp9SvihmJIVB41Jo1pPtEq_Cg/exec"

const name = ref<string>("")
const loading = ref<boolean>(true)
const error = ref<string>("")

function getIDFromHash(): string {
  const hash = window.location.hash
  const parts = hash.split("/")
  return parts[1] || ""
}

onMounted(async () => {
  try {
    const id = getIDFromHash()

    if (!id) {
      error.value = "Missing ID in URL"
      return
    }

    const res = await fetch(`${API}?id=${encodeURIComponent(id)}`)
    const data: ApiResponse = await res.json()

    if (data.ok) {
      name.value = data.row.Name
    } else {
      error.value = data.error
    }
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Unknown error"
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <div>
    <h2 v-if="loading">Loading...</h2>
    <p v-else-if="error">{{ error }}</p>
    <h1 v-else>Hello {{ name || "guest" }}</h1>
  </div>
</template>