<template>
    <div id="select-lang">
        <p v-if="select">סורק ב{{ selectlang.name }} בלבד <button @click="select = false">לשינוי</button></p>
        <div v-else>
            <template v-for="(lang, index) in language" :key="index">
                <label :for="lang.lang">{{ lang.name }}</label>
                <input type="radio" :id="lang.lang" :value="lang" v-model="selectlang">
            </template>
            <br />
            <button @click="select = true, $emit('setLanguage', selectlang)">שנה</button>
        </div>
    </div>
</template>

<script>
import { ref } from 'vue'
export default {
    setup() {
        const language = [{ lang: "heb", name: "עברית" }, { lang: "eng", name: "אנגלית" }, { lang: "eng+heb", name: "עברית ואנגלית" }]
        const selectlang = ref(language[0])
        const select = ref(true)
        return { language, selectlang, select }
    }

}
</script>

<style>
#select-lang {
    padding: 10px;
}
</style>