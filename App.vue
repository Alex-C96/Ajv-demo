<!-- App.vue -->
<template>
  <div class="container">
    <!-- Header Section -->
    <header class="header">
      <h1>JSON Schema Validator</h1>
      <p>Validate JSON data against JSON Schema using Ajv</p>
      <button @click="loadExample" class="btn-example">
        Load Example
      </button>
    </header>

    <!-- Input Grid: Schema and JSON side by side -->
    <div class="input-grid">
      <!-- JSON Schema Input -->
      <div class="input-section">
        <label for="schema-input">JSON Schema</label>
        <textarea
          id="schema-input"
          v-model="schema"
          placeholder="Paste your JSON schema here..."
          class="textarea"
        ></textarea>
      </div>

      <!-- JSON Data Input -->
      <div class="input-section">
        <label for="json-input">JSON Data</label>
        <textarea
          id="json-input"
          v-model="jsonData"
          placeholder="Paste your JSON data here..."
          class="textarea"
        ></textarea>
      </div>
    </div>

    <!-- Validate Button -->
    <div class="button-section">
      <button @click="validateJson" class="btn-validate">
        Validate JSON
      </button>
    </div>

    <!-- Results Section - Only shows when validation has been run -->
    <div v-if="validationResult" class="result-section" :class="resultClass">
      <div class="result-header">
        <span class="icon">{{ validationResult.valid ? '✓' : '✗' }}</span>
        <h2>{{ validationResult.valid ? 'Validation Passed!' : 'Validation Failed' }}</h2>
      </div>

      <!-- Error Details - Only shows if there are errors -->
      <div v-if="validationResult.errors && validationResult.errors.length > 0" class="errors">
        <h3>Errors:</h3>
        <ul>
          <!-- Loop through each error and display it -->
          <li v-for="(error, index) in validationResult.errors" :key="index">
            {{ error }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
// Import ref from Vue for reactive state
// ref() creates a reactive reference that Vue tracks for changes
import { ref } from 'vue';

// Import Ajv for JSON Schema validation
// Ajv is the fastest and most feature-complete JSON Schema validator
import Ajv from 'ajv';

// Define TypeScript interface for validation results
// This helps with type safety and IDE autocomplete
interface ValidationResult {
  valid: boolean;           // Whether validation passed
  errors?: string[];        // Array of error messages (optional)
}

// Create reactive state variables using ref<Type>()
// These variables will trigger re-renders when their .value changes
const schema = ref<string>('');      // Stores the JSON schema as a string
const jsonData = ref<string>('');    // Stores the JSON data as a string
const validationResult = ref<ValidationResult | null>(null);  // Stores validation results or null

// Computed property for CSS class based on validation result
// In Vue, you can use computed() for derived values, but here we'll use a simple getter
const resultClass = computed(() => {
  if (!validationResult.value) return '';
  return validationResult.value.valid ? 'result-success' : 'result-error';
});

// Example data to help users understand the format
const exampleSchema = `{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1
    },
    "age": {
      "type": "number",
      "minimum": 0,
      "maximum": 150
    },
    "email": {
      "type": "string",
      "format": "email"
    }
  },
  "required": ["name", "age"],
  "additionalProperties": false
}`;

const exampleData = `{
  "name": "John Doe",
  "age": 30,
  "email": "john@example.com"
}`;

// Function to load example data into the textareas
// Arrow functions are commonly used in Vue 3 Composition API
const loadExample = (): void => {
  schema.value = exampleSchema;
  jsonData.value = exampleData;
  // Clear previous results when loading new example
  validationResult.value = null;
};

// Main validation function using Ajv
const validateJson = (): void => {
  try {
    // Parse the schema string into a JavaScript object
    // JSON.parse() will throw an error if the JSON is invalid
    const parsedSchema = JSON.parse(schema.value);
    const parsedData = JSON.parse(jsonData.value);

    // Create a new Ajv instance
    // Ajv options can be configured here (allErrors, verbose, etc.)
    const ajv = new Ajv({ allErrors: true });  // allErrors: collect all errors, not just the first
    
    // Compile the schema into a validation function
    // This is efficient if you need to validate multiple times
    const validate = ajv.compile(parsedSchema);
    
    // Run the validation
    // validate() returns true/false and populates validate.errors if false
    const valid = validate(parsedData);

    // Format errors into readable strings
    // Ajv provides detailed error objects that we convert to messages
    const errors: string[] = [];
    if (!valid && validate.errors) {
      // Loop through each error from Ajv
      validate.errors.forEach((err) => {
        // Build a readable error message from Ajv's error object
        // err.instancePath shows where in the data the error occurred
        // err.message is the human-readable error description
        const path = err.instancePath || '/';  // Root path if empty
        const message = `${path}: ${err.message}`;
        errors.push(message);
        
        // You can also include additional details:
        // errors.push(`${path}: ${err.message} (${JSON.stringify(err.params)})`);
      });
    }

    // Update the reactive validationResult
    // Vue will automatically re-render the component with new data
    validationResult.value = {
      valid,
      errors: errors.length > 0 ? errors : undefined
    };

  } catch (error) {
    // Handle JSON parsing errors or Ajv errors
    // Type guard to safely access error message
    const errorMessage = error instanceof Error ? error.message : 'Unknown error occurred';
    
    // Set validation result to show the parsing error
    validationResult.value = {
      valid: false,
      errors: [`Parse error: ${errorMessage}`]
    };
  }
};

// Note: In Vue 3 Composition API with <script setup>:
// - No need to explicitly return variables/functions
// - Everything defined at top level is available in the template
// - Reactive state (.value) is automatically unwrapped in templates
</script>

<style scoped>
/* Scoped styles only apply to this component */

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.header {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.header h1 {
  margin: 0 0 0.5rem 0;
  color: #2c3e50;
}

.header p {
  margin: 0 0 1rem 0;
  color: #666;
}

.btn-example {
  background: #5c6bc0;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
}

.btn-example:hover {
  background: #3949ab;
}

.input-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

/* Responsive: stack on smaller screens */
@media (max-width: 768px) {
  .input-grid {
    grid-template-columns: 1fr;
  }
}

.input-section {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.input-section label {
  display: block;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: #2c3e50;
}

.textarea {
  width: 100%;
  height: 300px;
  padding: 1rem;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  resize: vertical;
}

.textarea:focus {
  outline: none;
  border-color: #5c6bc0;
}

.button-section {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 1.5rem;
}

.btn-validate {
  width: 100%;
  padding: 1rem;
  background: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
}

.btn-validate:hover {
  background: #45a049;
}

.result-section {
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.result-success {
  background: #e8f5e9;
  border: 2px solid #4caf50;
}

.result-error {
  background: #ffebee;
  border: 2px solid #f44336;
}

.result-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.icon {
  font-size: 2rem;
  font-weight: bold;
}

.result-success .icon {
  color: #4caf50;
}

.result-error .icon {
  color: #f44336;
}

.result-header h2 {
  margin: 0;
  color: #2c3e50;
}

.errors {
  margin-top: 1rem;
}

.errors h3 {
  margin: 0 0 0.5rem 0;
  color: #2c3e50;
}

.errors ul {
  margin: 0;
  padding-left: 1.5rem;
}

.errors li {
  margin: 0.5rem 0;
  color: #c62828;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
}
</style>

/* ============================================
   PACKAGE.JSON DEPENDENCIES
   ============================================
   
   You'll need to install these packages:
   
   {
     "dependencies": {
       "vue": "^3.3.0",
       "ajv": "^8.12.0"
     },
     "devDependencies": {
       "@vitejs/plugin-vue": "^4.0.0",
       "typescript": "^5.0.0",
       "vite": "^4.0.0",
       "vue-tsc": "^1.0.0"
     }
   }
   
   Install with: npm install vue ajv
   
   ============================================
   KEY VUE 3 + TYPESCRIPT CONCEPTS
   ============================================
   
   1. <script setup lang="ts">
      - Modern Vue 3 syntax (Composition API)
      - "setup" means this script runs during component setup
      - "lang='ts'" enables TypeScript
   
   2. ref<Type>()
      - Creates reactive state that Vue tracks
      - Access/modify with .value in script
      - Automatically unwrapped in template (no .value needed)
   
   3. v-model="variable"
      - Two-way data binding
      - Syncs textarea value with reactive state
      - Changes in either direction update the other
   
   4. @click="function"
      - Shorthand for v-on:click
      - Calls function when element is clicked
   
   5. v-if="condition"
      - Conditional rendering
      - Element only exists in DOM if condition is true
   
   6. v-for="item in array"
      - List rendering
      - Loops through array and creates element for each item
      - :key is required for performance/tracking
   
   7. :class="dynamicClass"
      - Dynamic class binding
      - Applies CSS classes based on JavaScript expressions
   
   8. TypeScript Interfaces
      - Define shape of objects for type safety
      - Helps catch errors at compile time
      - Provides better IDE autocomplete
   
   ============================================
*/
