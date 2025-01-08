<template>
  <div class="hamming-detector">
    <div class="detector-header">
      <h1>Group 1: Hamming Code Parity Detector</h1>
      <p>Step 1: Calculate check bits for stored and fetched words</p>
    </div>

    <div class="input-container">
      <div class="input-group">
        <label for="storedWord">Word Stored (8 bits for D8D7D6D5D4D3D2D1):</label>
        <input 
          id="storedWord"
          v-model="storedWord"
          type="text"
          placeholder="Enter 8 bits (e.g., 10110011)"
          @input="validateAndUpdateDisplay"
        >
      </div>

      <div class="input-group">
        <label for="fetchedWord">Word Fetched (8 bits for D8D7D6D5D4D3D2D1):</label>
        <input 
          id="fetchedWord"
          v-model="fetchedWord"
          type="text"
          placeholder="Enter 8 bits (e.g., 10110011)"
          @input="validateAndUpdateDisplay"
        >
      </div>

      <button 
        @click="calculateAndCheck"
        :disabled="!isValidInput"
        class="check-button"
      >
        Calculate and Check Parity
      </button>
    </div>

    <div class="calculations-container">
      <div class="step-container">
        <h3>Input Visualization</h3>
        <table class="bit-table">
          <tbody>
            <tr>
              <td class="row-header">Bit position</td>
              <td v-for="n in 12" :key="'pos-'+n">{{ 13-n }}</td>
            </tr>
            <tr>
              <td class="row-header">Check bit</td>
              <td v-for="n in 12" :key="'check-'+n">
                {{ getCheckBitLabel(13-n) }}
              </td>
            </tr>
            <tr>
              <td class="row-header">Data bit</td>
              <td v-for="n in 12" :key="'data-'+n">
                {{ getDataBitLabel(13-n) }}
              </td>
            </tr>
            <tr>
              <td class="row-header">Word stored</td>
              <td v-for="(bit, index) in displayedStoredWord" :key="'stored-'+index">
                {{ bit }}
              </td>
            </tr>
            <tr>
              <td class="row-header">Word fetched</td>
              <td v-for="(bit, index) in displayedFetchedWord" :key="'fetched-'+index">
                {{ bit }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-if="showCalculations">
        <div class="step-container">
          <h3>Step 2: Calculate Check Bits for Word Stored</h3>
          <div class="calculation-steps">
            <div class="calc-row" v-for="calc in storedCalculations" :key="calc.checkBit">
              <span class="check-bit">{{ calc.checkBit }}</span>
              <span>=</span>
              <template v-for="(bit, index) in calc.dataBits" :key="index">
                <span class="data-bit">{{ bit }}</span>
                <span v-if="index < calc.dataBits.length - 1">⊕</span>
              </template>
              <span>=</span>
              <span class="result">{{ calc.result }}</span>
            </div>
          </div>
        </div>

        <div class="step-container">
          <h3>Step 3: Calculate Check Bits for Word Fetched</h3>
          <div class="calculation-steps">
            <div class="calc-row" v-for="calc in fetchedCalculations" :key="calc.checkBit">
              <span class="check-bit">{{ calc.checkBit }}</span>
              <span>=</span>
              <template v-for="(bit, index) in calc.dataBits" :key="index">
                <span class="data-bit">{{ bit }}</span>
                <span v-if="index < calc.dataBits.length - 1">⊕</span>
              </template>
              <span>=</span>
              <span class="result">{{ calc.result }}</span>
            </div>
          </div>
        </div>

        <div class="step-container">
          <h3>Step 4: Check Parity Differences</h3>
          <div class="parity-check">
            <div v-for="check in parityComparison" :key="check.bit" class="parity-row">
              <span class="check-bit">{{ check.bit }}</span>
              <span>Stored: {{ check.stored }}</span>
              <span>Fetched: {{ check.fetched }}</span>
              <span :class="['status', check.matches ? 'match' : 'error']">
                {{ check.matches ? 'Match' : 'Error' }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'HammingDetector',
  data() {
    return {
      storedWord: '',
      fetchedWord: '',
      error: '',
      isValidInput: false,
      showCalculations: false,
      displayedStoredWord: Array(12).fill('-'),
      displayedFetchedWord: Array(12).fill('-'),
      storedCalculations: [],
      fetchedCalculations: [],
      parityComparison: []
    }
  },
  methods: {
    validateAndUpdateDisplay() {
      this.error = '';
      this.showCalculations = false;
      
      const binaryRegex = /^[01]{8}$/;
      
      if (this.storedWord && !binaryRegex.test(this.storedWord)) {
        this.error = 'Word stored must be exactly 8 bits (0s and 1s)';
        this.isValidInput = false;
        return;
      }
      
      if (this.fetchedWord && !binaryRegex.test(this.fetchedWord)) {
        this.error = 'Word fetched must be exactly 8 bits (0s and 1s)';
        this.isValidInput = false;
        return;
      }

      // Update displayed words
      this.updateDisplayedWords();
      
      this.isValidInput = this.storedWord && this.fetchedWord;
    },

    updateDisplayedWords() {
      // Create arrays of 12 positions filled with '-'
      this.displayedStoredWord = Array(12).fill('-');
      this.displayedFetchedWord = Array(12).fill('-');

      if (this.storedWord.length === 8) {
        // Map 8-bit input to 12-bit display (D8->D1)
        const positions = [12, 11, 10, 9, 7, 6, 5, 3];
        for (let i = 0; i < 8; i++) {
          this.displayedStoredWord[12 - positions[i]] = this.storedWord[i];
        }
      }

      if (this.fetchedWord.length === 8) {
        // Map 8-bit input to 12-bit display (D8->D1)
        const positions = [12, 11, 10, 9, 7, 6, 5, 3];
        for (let i = 0; i < 8; i++) {
          this.displayedFetchedWord[12 - positions[i]] = this.fetchedWord[i];
        }
      }
    },

    getCheckBitLabel(position) {
      const checkBits = { 1: 'C1', 2: 'C2', 4: 'C4', 8: 'C8' };
      return checkBits[position] || '';
    },

    getDataBitLabel(position) {
      const dataBits = {
        12: 'D8', 11: 'D7', 10: 'D6', 9: 'D5',
        7: 'D4', 6: 'D3', 5: 'D2', 3: 'D1'
      };
      return dataBits[position] || '';
    },

    calculateCheckBits(word) {
      const calculations = [];
      
      const checkBitDefinitions = {
        C1: { bits: ['D1', 'D2', 'D4', 'D5', 'D7'] },
        C2: { bits: ['D1', 'D3', 'D4', 'D6', 'D7'] },
        C4: { bits: ['D2', 'D3', 'D4', 'D8'] },
        C8: { bits: ['D5', 'D6', 'D7', 'D8'] }
      };

      for (const [checkBit, def] of Object.entries(checkBitDefinitions)) {
        const dataBits = def.bits.map(dbit => {
          const index = parseInt(dbit.substring(1)) - 1;
          return `${dbit}(${word[7-index]})`;
        });
        
        const result = def.bits.map(dbit => {
          const index = parseInt(dbit.substring(1)) - 1;
          return parseInt(word[7-index]);
        }).reduce((a, b) => a ^ b, 0);

        calculations.push({
          checkBit,
          dataBits,
          result: result.toString()
        });
      }

      return calculations;
    },

    calculateAndCheck() {
      this.storedCalculations = this.calculateCheckBits(this.storedWord);
      this.fetchedCalculations = this.calculateCheckBits(this.fetchedWord);

      this.parityComparison = ['C1', 'C2', 'C4', 'C8'].map(bit => {
        const stored = this.storedCalculations.find(c => c.checkBit === bit).result;
        const fetched = this.fetchedCalculations.find(c => c.checkBit === bit).result;
        return {
          bit,
          stored,
          fetched,
          matches: stored === fetched
        };
      });

      this.showCalculations = true;
    }
  }
}
</script>

<style scoped>
.hamming-detector {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
  background-color: #1e1e1e;
  color: #d4d4d4;
  font-family: 'Consolas', monospace;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.detector-header {
  text-align: center;
  margin-bottom: 30px;
  width: 100%;
}

.calculations-container {
  width: 100%;
  margin-top: 20px;
}

.input-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin: 20px 0;
  width: 100%;
  align-items: center;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
  width: 100%;
  max-width: 500px;
}

/* Rest of the styles remain the same */
.step-container {
  background-color: #252526;
  padding: 15px;
  margin-bottom: 20px;
  border-radius: 4px;
  width: 100%;
}

.calculation-steps {
  margin-top: 15px;
  padding: 10px;
  background-color: #1e1e1e;
  border-radius: 4px;
}

.calc-row {
  display: flex;
  gap: 8px;
  align-items: center;
  margin: 8px 0;
  font-family: monospace;
}

.check-bit {
  color: #4ec9b0;
  font-weight: bold;
}

.data-bit {
  color: #9cdcfe;
}

.result {
  color: #ce9178;
  font-weight: bold;
}

.bit-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
  background-color: #252526;
}

.bit-table td {
  border: 1px solid #404040;
  padding: 8px;
  text-align: center;
}

.row-header {
  background-color: #2d2d2d;
  color: #9cdcfe;
  font-weight: bold;
  text-align: right;
  padding-right: 15px;
}

input {
  padding: 8px;
  background-color: #3c3c3c;
  border: 1px solid #404040;
  border-radius: 4px;
  color: #d4d4d4;
  font-family: 'Consolas', monospace;
}

.check-button {
  padding: 10px;
  background-color: #0e639c;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  width: 200px;
}

.check-button:disabled {
  background-color: #404040;
  cursor: not-allowed;
}

.parity-check {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.parity-row {
  display: flex;
  gap: 20px;
  align-items: center;
}

.status {
  padding: 2px 8px;
  border-radius: 3px;
}

.status.match {
  background-color: #1e4620;
  color: #4ec9b0;
}

.status.error {
  background-color: #4e1f1f;
  color: #f48771;
}

.error-message {
  color: #f48771;
  margin: 10px 0;
  text-align: center;
}
</style>