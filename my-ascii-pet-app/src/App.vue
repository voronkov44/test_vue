<template>
  <div class="container">
    <div class="header">
      <h1 class="title">ASCII - PET</h1>
    </div>

    <div class="display-card">
      <el-icon class="delete-icon" @click="deletePet">
        <Delete />
      </el-icon>
      <div class="label">ASCII - картинка</div>
      <div class="ascii-display">
        <pre>{{ savedAscii || '.........／＞　 フ...........\n　　　　　| 　_　 _|\n　 　　　／ミ _x 彡\n　　 　 /　　　 　 |\n　　　 /　 ヽ　　 ﾉ\n　／￣|　　 |　|　|\n　| (￣ヽ＿_ヽ_)_)\n　＼二つ' }}</pre>
        <el-icon class="copy-icon" @click="copyText(savedAscii)">
          <CopyDocument />
        </el-icon>
      </div>

      <div class="label">Описание питомца</div>
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

      <!-- Переключатель валидации -->
      <div style="margin: 10px 0;">
        <el-switch
            v-model="strictValidation"
            active-text="Строгая валидация"
            inactive-text="Гибкая валидация"
        />
      </div>

      <el-button type="primary" class="save-button" @click="savePet">Сохранить</el-button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { CopyDocument, Delete } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'

const asciiArt = ref('')
const description = ref('')
const savedAscii = ref('')
const savedDescription = ref('')
const strictValidation = ref(true) // параметр для переключения режима валидации

// Функция для сохранения данных (метод PUT)
const savePet = async () => {
  const ascii = asciiArt.value.trim()
  const desc = description.value.trim()

  // Валидация ASCII (строгая)
  const isValidAscii = (str) => {
    for (let i = 0; i < str.length; i++) {
      const code = str.charCodeAt(i)
      if (code !== 10 && (code < 32 || code > 126)) {
        return false
      }
    }
    return true
  }

  // Валидация ASCII (гибкая для арт ASCII)
  const isPrintable = (input) =>
      /^[\p{L}\p{N}\p{P}\p{S}\p{Z}\n\r\t]*$/u.test(input)

  if (strictValidation.value) {
    // строгая валидация
    if (!ascii) {
      ElMessage.error('Введите ASCII картинку!')
      return
    }
    if (!isValidAscii(ascii)) {
      ElMessage.error('ASCII картинка должна содержать только допустимые символы!')
      return
    }
  } else {
    // гибкая валидация
    if (!ascii) {
      ElMessage.error('Введите ASCII картинку!')
      return
    }
    if (!isPrintable(ascii)) {
      ElMessage.error('Картинка содержит недопустимые символы!')
      return
    }
  }

  if (ascii.length > 500) {
    ElMessage.error('ASCII картинка не должна превышать 500 символов!')
    return
  }

  if (!desc) {
    ElMessage.error('Введите описание питомца!')
    return
  }

  if (desc.length > 200) {
    ElMessage.error('Описание не должно превышать 200 символов!')
    return
  }

  const petData = {
    ascii: ascii,
    description: desc
  }

  try {
    const response = await fetch('http://localhost:8080/v1/pet', {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(petData)
    })

    if (response.ok) {
      console.log('Pet saved successfully')
      await response.text()

      savedAscii.value = ascii
      savedDescription.value = desc

      asciiArt.value = ''
      description.value = ''
    } else {
      const errorText = await response.text()
      console.error('Failed to save pet:', errorText)
      ElMessage.error(`Ошибка сохранения: ${errorText}`)
    }
  } catch (error) {
    console.error('Error:', error)
    ElMessage.error('Ошибка соединения с сервером')
  }
}

// Функция для копирования текста
const copyText = (text) => {
  navigator.clipboard.writeText(text).then(() => {
    console.log('Text copied to clipboard')
  })
}

// Функция для загрузки pet (метод GET)
const loadPet = async () => {
  try {
    const response = await fetch('http://localhost:8080/v1/pet')

    if (response.ok) {
      const data = await response.json()
      console.log('Loaded pet:', data)

      savedAscii.value = data.ascii
      savedDescription.value = data.description
    } else if (response.status === 204) {
      console.log('No pet data found')
    } else {
      console.error('Failed to load pet')
    }
  } catch (error) {
    console.error('Error loading pet:', error)
  }
}

// загружаем pet при загрузке страницы
window.addEventListener('load', loadPet)

const deletePet = async () => {
  try {
    const response = await fetch('http://localhost:8080/v1/pet', {
      method: 'DELETE'
    })

    if (response.status === 204) {
      console.log('Pet deleted successfully')
      savedAscii.value = ''
      savedDescription.value = ''
    } else {
      console.error('Failed to delete pet')
    }
  } catch (error) {
    console.error('Error deleting pet:', error)
  }
}
</script>
