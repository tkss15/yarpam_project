<!-- App.vue -->
<template>
  <div class="app">
    <header>
      <h1>בדיקה יומית - צוות {{ teamNumber }}</h1>
    </header>
    <main>
      <form @submit.prevent="submitForm">
        <section class="basic-info">
          <h2>מידע בסיסי</h2>
          <div class="form-group">
            <label for="date">תאריך:</label>
            <input type="date" id="date" v-model="formData.date" required>
          </div>
          <div class="form-group">
            <label for="seniorCaregiver">שם מטפל בכיר:</label>
            <input type="text" id="seniorCaregiver" v-model="formData.seniorCaregiver" required>
          </div>
          <div class="form-group">
            <label for="ambulanceInfo">מספר וסוג אמבולנס:</label>
            <input type="text" id="ambulanceInfo" v-model="formData.ambulanceInfo" required>
          </div>
        </section>
        
        <section class="items-list">
          <h2>רשימת פריטים</h2>
          <div class="table-header">
            <div class="col">שם</div>
            <div class="col">תקן</div>
            <div class="col">קיים/חוסר</div>
          </div>
          <div v-for="(item, index) in items" :key="index" class="table-row">
            <div class="col">{{ item.name }}</div>
            <div class="col">{{ item.standard }}</div>
            <div class="col">
              <template v-if="item.standard === 0 || item.standard === 1">
                <div class="toggle-switch">
                  <input
                    type="checkbox"
                    :id="`actual-${index}`"
                    v-model="item.actual"
                    :true-value="1"
                    :false-value="0"
                  >
                  <label :for="`actual-${index}`" class="toggle-label"></label>
                </div>
              </template>
              <input
                v-else
                type="number"
                :id="`actual-${index}`"
                v-model="item.actual"
                min="0"
              >
            </div>
          </div>
        </section>
        
        <section class="notes-section">
          <h2>הערות</h2>
          <div v-for="(item, index) in items" :key="index">
            <div v-if="item.standard != item.actual" class="note-item">
              <h3>{{ item.name }}</h3>
              <div class="input-group">
                <label :for="`notes-${index}`">הערות:</label>
                <input type="text" :id="`notes-${index}`" v-model="item.notes">
              </div>
              <div class="input-group">
                <label :for="`reason-${index}`">סיבה לפער:</label>
                <input type="text" :id="`reason-${index}`" v-model="item.reason">
              </div>
            </div>
          </div>
        </section>
        
        <button type="submit" class="submit-btn">שלח טופס</button>
      </form>
    </main>
  </div>
</template>

<script>
import { ref, computed, watchEffect } from 'vue';
import { useRoute } from 'vue-router';

export default {
  setup() {
    const route = useRoute();
    const teamNumber = ref(1); // Default to team 1
    watchEffect(() => {
      const pathTeamNumber = parseInt(route.params.teamNumber);
      if (!isNaN(pathTeamNumber)) {
        teamNumber.value = pathTeamNumber;
      }
    })

    const formData = ref({
      date: new Date().toISOString().substr(0, 10),
      seniorCaregiver: '',
      ambulanceInfo: '',
    });

    const items = ref([
      { name: 'אלונקות', standard: 4, actual: 4, notes: '', reason: '' },
      { name: 'בלון חמצן גדול', standard: 1, actual: 0, notes: '', reason: '' },
      { name: 'וַסָּת חמצן לקיר', standard: 1, actual: 0, notes: '', reason: '' },
      // Add all other items here...
    ]);

    const incompleteItems = computed(() => {
      return items.value.filter(item => 
        item.standard !== item.actual || 
        (item.standard !== item.actual && !item.reason)
      );
    });

    const submitForm = () => {
      if (incompleteItems.value.length > 0) {
        alert('נא למלא את כל הפרטים הנדרשים עבור פריטים עם פערים.');
        return;
      }
      console.log('Form submitted:', formData.value, items.value);
      // Here you would typically send the data to a server
      alert('הטופס נשלח בהצלחה!');
    };

    return {
      teamNumber,
      formData,
      items,
      submitForm,
    };
  },
}
</script>

<style scoped>
.app {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  direction: rtl;
}

header {
  text-align: center;
  margin-bottom: 20px;
}

h1, h2, h3 {
  margin-bottom: 15px;
}

.form-group, .input-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input[type="text"],
input[type="date"],
input[type="number"] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

.table-header, .table-row {
  display: flex;
  border-bottom: 1px solid #ddd;
  padding: 10px 0;
}

.table-header {
  font-weight: bold;
  background-color: #f0f0f0;
}

.col {
  flex: 1;
  padding: 0 5px;
  display: flex;
  align-items: center;
}

.toggle-switch {
  position: relative;
  display: inline-block;
  width: 60px;
  height: 34px;
}

.toggle-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-label {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 34px;
}

.toggle-label:before {
  position: absolute;
  content: "";
  height: 24px;
  width: 24px;
  left: 4px;
  bottom: 3px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

input:checked + .toggle-label {
  background-color: #2196F3;
}

input:checked + .toggle-label:before {
  transform: translateX(26px);
}

.notes-section {
  margin-top: 30px;
}

.note-item {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 20px;
}

.submit-btn {
  background-color: #4CAF50;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 18px;
  width: 100%;
  margin-top: 20px;
}

.submit-btn:hover {
  background-color: #45a049;
}

@media (max-width: 600px) {
  .col {
    font-size: 14px;
  }
  
  input[type="text"],
  input[type="date"],
  input[type="number"] {
    font-size: 14px;
  }
  
  .submit-btn {
    font-size: 16px;
  }
}
</style>