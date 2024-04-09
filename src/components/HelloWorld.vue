<script setup>
import { ref, computed } from "vue"
import dayjs from "dayjs"
import axios from "axios"

axios.defaults.baseURL =
  import.meta.API_URL || "https://web-tier-02-was.swd.com"

const loading = ref(false)
const data = ref([])
const input = ref("")
const inputRef = ref(null)
const boolData = computed(() => data.value.filter(Boolean))

async function fetchData() {
  loading.value = true
  const { data: listData } = await axios.get("/api/p")
  data.value = listData
  loading.value = false
}

async function writeData(text) {
  loading.value = true
  await axios.post("/api/p", { text: input.value })
  loading.value = false
}

fetchData()

async function onKeyPress(e) {
  if (!input.value || loading.value) return
  if (e.key === "Enter") {
    await writeData(input.value)
    await fetchData()
    input.value = ""
    inputRef.value?.focus()
  }
}

async function clearList() {
  loading.value = true
  await axios.post("/api/clear")
  await fetchData()
  loading.value = false
}
</script>

<template>
  <div class="w-full h-screen overflow-hidden flex justify-center items-center">
    <div class="text-center space-y-4">
      <input
        v-model="input"
        ref="inputRef"
        type="text"
        placeholder="Enter..."
        class="text-xl px-4 py-2 rounded"
        :disabled="loading"
        @keypress="onKeyPress"
      />
      <div>
        <button
          class="rounded-lg border border-gray-200 px-4 py-2 transition-colors hover:bg-gray-50"
          @click="clearList"
        >
          Clear
        </button>
      </div>
      <div class="max-h-[700px] overflow-y-auto">
        <table class="">
          <thead>
            <tr>
              <th class="min-w-[300px]">Message</th>
              <th>Created At</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="message in boolData">
              <td>
                <span class="truncate">{{ message.text }}</span>
              </td>
              <td>
                {{
                  dayjs(new Date(message.createdAt)).format(
                    "YYYY-MM-DD HH:mm:ss"
                  )
                }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
