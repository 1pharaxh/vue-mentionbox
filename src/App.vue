<template>
  <div>
    <input v-if="!ready" placeholder="Mention here!" v-model="inputValue" @keydown="handleKeydown" @mousedown="handleMousedown" style="width:400px; height:50px" />
    <ul v-if="filteredOptions.length > 0" class="dropdown-list">
      <li
        v-for="(option, index) in filteredOptions"
        :key="option"
        :class="{ 'selected': index === selectedOptionIndex }"
        @mousedown="selectOption(index)"
      >
        <span v-if="index === selectedOptionIndex" class="highlighted">{{ option }}</span>
        <span v-else>{{ option }}</span>
      </li>
    </ul>
    <div id="buttonDiv" v-if="ready">
      <button @click="generateNew">Generate!</button>
    </div>
    <div v-if="ready">
    <h2>Please Generate...</h2>
    </div>
    <div v-else>
    <h3>Please start typing</h3>
    <p> Use @ to mention and reload the page to unhide the generate button!</p>
    </div>
  </div>
</template>
<script>
export default {
  name: 'vue-mentions',
  
  data() {
    return {
      options: [],
      ready: true,
      inputValue: '',
      filteredOptions: [],
      selectedOptionIndex: -1,
    };
  },
  methods: {
    async generateNew() {
      let response = await fetch('http://mentionbox.onrender.com/generate');
      let data = await response;
      if (data.status == 200) {
        response = await fetch('http://mentionbox.onrender.com/fetch');
        data = await response.json();
        let obj = {};

        data.forEach(item => {
  let data = item._source.body;
  let parsedData = JSON.parse(data);

  let middle = Math.floor(parsedData.length / 2);

  parsedData.forEach((person, index) => {
    let name = person.name1.trimStart();
    let email = person.email.trimStart();
    let type = index < middle ? 'employee' : 'customer';

    obj[name] = [email, type];
  });
});

this.options = Object.keys(obj);
console.log(this.options);
if (this.options.length > 0) {
  this.ready = false;
}
      }
      
    },
    handleKeydown(event) {
      if(this.inputValue.includes('@')) {
        
        let temp = this.inputValue.split("@").pop();
        this.filteredOptions = this.options.filter((option) =>
        option.toLowerCase().startsWith(temp.toLowerCase())
      );
      if (event.key === 'Enter') {
        // Select the highlighted option when the enter key is pressed
        if (this.selectedOptionIndex >= 0) {
          this.selectOption(this.selectedOptionIndex);
        }
      }
      // Handle up and down arrow keys to navigate the dropdown menu
      if (event.key === 'ArrowUp') {
        event.preventDefault();
        this.selectedOptionIndex = Math.max(0, this.selectedOptionIndex - 1);
      } else if (event.key === 'ArrowDown') {
        event.preventDefault();
        this.selectedOptionIndex = Math.min(
          this.filteredOptions.length - 1,
          this.selectedOptionIndex + 1
        );
      }
      }
      // Filter the list of options based on the input value
      
    },
    handleMousedown() {
      // Reset the selected option index when the input field is clicked
      this.selectedOptionIndex = -1;
    },
    selectOption(index) {
      // Update the input field value with the selected option
      let temp = this.inputValue.split("@").shift();
      temp += this.filteredOptions[index];
      this.inputValue = temp;

      // Reset the filtered options and selected option index
      this.filteredOptions = [];
      this.selectedOptionIndex = -1;
    },
  },
};
</script>
<style>
.highlighted {
  background-color: #eee;
}

.dropdown-list {
  background-color: white;
  border: 1px solid #eee;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  list-style: none;
  margin: 0;
  padding: 8px 0;
  position: absolute;
  z-index: 1;
}

.dropdown-list li {
  cursor: pointer;
  padding: 8px 12px;
}

.dropdown-list li:hover {
  background-color: #eee;
}

.dropdown-list .selected {
  background-color: #f5f5f5;
}
.highlight {
    background-color: yellow;
    color:red;
  }
</style>