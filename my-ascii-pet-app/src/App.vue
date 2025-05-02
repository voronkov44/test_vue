<template>
  <div class="container">
    <div class="header">
      <h1 class="title">ASCII - PET</h1>
    </div>

    <div class="display-card">
      <div class="ascii-display">
        <pre>{{ savedAscii || '.........／＞　 フ...........\n　　　　　| 　_　 _|\n　 　　　／ミ _x 彡\n　　 　 /　　　 　 |\n　　　 /　 ヽ　　 ﾉ\n　／￣|　　 |　|　|\n　| (￣ヽ＿_ヽ_)_)\n　＼二つ' }}</pre>
        <el-icon class="copy-icon" @click="copyText(savedAscii)">
          <CopyDocument />
        </el-icon>
      </div>

      <div class="description-display">
        <span>{{ savedDescription || 'Это грустный котик - Степик\nПочему он грустный?\nПотому что тут нету картинки и описания...' }}</span>
        <el-icon class="copy-icon" @click="copyText(savedDescription)">
          <CopyDocument />
        </el-icon>
      </div>
    </div>

    <div class="form-card">
      <textarea
          v-model="asciiArt"
          class="ascii-input"
          placeholder="Вставьте сюда вашу ASCII картинку..."
          rows="8"
      ></textarea>

      <textarea
          v-model="description"
          class="description-input"
          placeholder="Описание питомца..."
          rows="2"
      ></textarea>

      <el-button type="primary" class="save-button" @click="savePet">Сохранить</el-button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { CopyDocument } from '@element-plus/icons-vue'

const asciiArt = ref('')
const description = ref('')
const savedAscii = ref('')
const savedDescription = ref('')

function savePet() {
  savedAscii.value = asciiArt.value
  savedDescription.value = description.value
  asciiArt.value = ''
  description.value = ''
}

function copyText(text) {
  if (!text) return
  navigator.clipboard.writeText(text)
}
</script>
