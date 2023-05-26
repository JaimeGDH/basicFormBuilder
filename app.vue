<script setup>
const axios = useNuxtApp().$axios;

const formSchema = ref(null);

const getForm = () => {
  try {
    axios.get('https://run.mocky.io/v3/6bd01347-72e9-49da-809a-d5002ca63b2c').then((response)=>{
      formSchema.value = response.data;
    })
  } catch (error) {
    console.error(`Hubo un error al obtener el formulario: ${ error }`);
  }  
};

const whatComponent = (schemaItem) => {
  switch (schemaItem.type) {
    case 'SELECT':
      return 'v-select'; 
    case 'INPUT':
      return 'v-text-field';
    case 'CHECKBOX':
      return 'v-checkbox'; 
    default:
      return schemaItem.form?.type || null;
  }
};

const handleItems = (schemaItem) => {
  switch (schemaItem.type) {
    case 'SELECT':
      return schemaItem.items.map(item => item.value);
    case 'INPUT':
      return 'v-text-field';
    default:
      return schemaItem.form?.type || null;
  }
};

const responses = ref(null);

const getResponses = () => {
  responses.value = formSchema.value.formulario.form.map(item => {
  return {
    question: item.question,
    response: item.response
  };   
});
}

onMounted(()=> {
  getForm();
})
</script>

<template>
  <v-card 
    v-if="formSchema"
    class="mx-auto form-container"
    max-width="500"
  >
    <template v-slot:title>
      {{  formSchema.formulario?.name  }}
    </template>
    <template v-slot:subtitle>
      {{ formSchema.formulario?.descripcion }}
    </template>
    <div
      v-for="(item, i) in formSchema.formulario?.form"
      :key="i"
    >     
      <v-form > 
        <component
          v-if="item.type !== 'CHECKBOX' && item.type !== 'RADIOBUTTON'"
          :label="item.question"
          :is="whatComponent(item)"
          :items="handleItems(item)"
          v-model="item.response"
        />
        <div v-if="item.type === 'CHECKBOX'">
          {{ item.question }}
          <v-checkbox
            v-for="checkboxItem in item.items"
            :key="checkboxItem"
            :value="checkboxItem"
            :label="checkboxItem"
            v-model="item.response"
          />
        </div>
        <div v-if="item.type === 'RADIOBUTTON'">
          {{ item.question }}
          <v-radio-group v-model="item.response">
            <v-radio 
              v-for="item in item.items"  
              :value="item"
              :label="item"        
            />
          </v-radio-group>
        </div>
      </v-form>
    </div>
    <v-card-actions>
      <v-btn color="success" @click="getResponses()">
        Confirmar
      </v-btn>
    </v-card-actions>

    <v-table v-if="responses">
    <thead>
      <tr>
        <th class="text-left">
          Pregunta
        </th>
        <th class="text-left">
          Respuesta
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="item in responses"
        :key="item.question"
      >
        <td>{{ item.question }}</td>
        <td>{{ item.response }}</td>
      </tr>
    </tbody>
  </v-table>
  </v-card>  
</template>

<style>
.content-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-container {
  margin-top: 50px;
}
</style>