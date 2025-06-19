<template>
  <div class="main-container">

    <div v-if="appState === 'splash'" class="splash-screen">
      <p class="line1">INITIALIZING HOLOCULATOR OS v3.14...</p>
      <p class="line2">SYSTEM BOOT SEQUENCE INITIATED...</p>
    </div>

    <div v-else-if="appState === 'login'" class="login-screen">
      <h1>AWAITING AUTHENTICATION</h1>
      <p>ENTER ACCESS CODE:</p>
      <input 
        type="password"
        v-model="passwordInput"
        @keyup.enter="checkPassword"
        :class="{ 'error': loginError }"
        placeholder="_"
      />
      <p v-if="loginError" class="error-message">ACCESS DENIED</p>
    </div>

    <div v-else-if="appState === 'loading'" class="loading-screen">
      <h1>DECRYPTING CORE MATRIX</h1>
      <ul class="system-check">
        <li v-for="(item, index) in loadingChecks" :key="index" :class="{ 'visible': index < loadingProgress }">
          <span class="status">{{ item.status }}</span> {{ item.text }}
        </li>
      </ul>
    </div>

    <Transition name="fade">
      <div v-if="appState === 'ready'" class="holoculator-wrapper">
        <div class="history">
          <div class="history-title">LOG</div>
          <div v-for="(item, index) in history" :key="index">{{ item }}</div>
        </div>
        <div class="calculator">
          <div class="display">
            <span class="scanline"></span>
            {{ current || '0' }}
          </div>
          <div @click="clear" class="btn func">C</div><div @click="sign" class="btn func">+/-</div><div @click="percent" class="btn func">%</div><div @click="setOperator('/')" class="btn operator">÷</div>
          <div @click="append('7')" class="btn">7</div><div @click="append('8')" class="btn">8</div><div @click="append('9')" class="btn">9</div><div @click="setOperator('*')" class="btn operator">x</div>
          <div @click="append('4')" class="btn">4</div><div @click="append('5')" class="btn">5</div><div @click="append('6')" class="btn">6</div><div @click="setOperator('-')" class="btn operator">-</div>
          <div @click="append('1')" class="btn">1</div><div @click="append('2')" class="btn">2</div><div @click="append('3')" class="btn">3</div><div @click="setOperator('+')" class="btn operator">+</div>
          <div @click="append('0')" class="btn zero">0</div><div @click="dot" class="btn">.</div><div @click="equal" class="btn operator">=</div>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { supabase } from '../lib/supabaseClient' 
import { ref, onMounted, watch } from 'vue';

// --- STATE MANAGEMENT ---
// Beheert welke fase / welk scherm van de app we zien
const appState = ref('splash'); // 'splash', 'login', 'loading', 'ready'
const passwordInput = ref('');
const loginError = ref(false);
const loadingProgress = ref(0);

// --- LIFECYCLE HOOKS ---
// onMounted wordt uitgevoerd zodra het component voor het eerst wordt geladen
onMounted(() => {
  // De timer om van splash naar login te gaan
  setTimeout(() => {
    appState.value = 'login';
  }, 4000); 

  // NIEUW: Haal de bestaande geschiedenis op als de app laadt
  fetchCalculations();
});

// watch luistert naar veranderingen in een ref.
// Zodra de appState 'loading' wordt, starten we de laad-animatie en de timer.
watch(appState, (newState) => {
  if (newState === 'loading') {
    // Start de animatie voor de system check
    const interval = setInterval(() => {
      loadingProgress.value++;
    }, 3000); // Elke 3 seconden een nieuw item

    // Start de timer om na 20 seconden naar de rekenmachine te gaan
    setTimeout(() => {
      clearInterval(interval); // Stop de animatie-interval
      appState.value = 'ready';
    }, 20000); // 20 seconden
  }
});

// --- FUNCTIES ---
const checkPassword = () => {
  if (passwordInput.value === 'Nova') {
    loginError.value = false;
    appState.value = 'loading'; // Ga naar de laadfase
  } else {
    loginError.value = true;
    passwordInput.value = '';
    // Haal de error na een seconde weer weg
    setTimeout(() => {
      loginError.value = false;
    }, 1000);
  }
};

const loadingChecks = [
  { text: 'Quantum Core Calibration', status: 'OK' },
  { text: 'Neural Network Sync', status: 'OK' },
  { text: 'Reality Distortion Fields', status: 'OK' },
  { text: 'Subspace Comms Link', status: 'OK' },
  { text: 'Flux Capacitor Charge', status: 'OK' },
  { text: 'Finalizing Boot Sequence', status: 'OK' },
];

const fetchCalculations = async () => {
  try {
    const { data, error } = await supabase
      .from('calculations')          // Vanuit de 'calculations' tabel
      .select('calculation_string')  // Selecteer alleen de kolom met de tekst
      .order('created_at', { ascending: false }) // Sorteer op aanmaakdatum, nieuwste eerst
      .limit(5);                     // Haal maximaal de laatste 5 op

    if (error) throw error;

    // Als er data is, zet die in onze 'history' ref
    if (data) {
      // .map() zorgt ervoor dat we een schone lijst van alleen de tekst krijgen
      history.value = data.map(item => item.calculation_string);
    }
  } catch (error) {
    console.error('Fout bij ophalen geschiedenis:', error.message);
  }
};


// --- REKENMACHINE LOGICA (ongewijzigd) ---
const current = ref('');
const previous = ref(null);
const operator = ref(null);
const operatorClicked = ref(false);
const history = ref([]);

const clear = () => { current.value = ''; previous.value = null; operator.value = null; operatorClicked.value = false; };
const sign = () => { if (current.value) { current.value = current.value.charAt(0) === '-' ? current.value.slice(1) : `-${current.value}`; } };
const percent = () => { if (current.value) { current.value = `${parseFloat(current.value) / 100}`; } };
const append = (number) => { if (operatorClicked.value) { current.value = ''; operatorClicked.value = false; } current.value = `${current.value}${number}`; if (current.value === '777') { current.value = '>>> HACKED <<<'; } };
const dot = () => { if (!current.value.includes('.')) { append('.'); } };
const setOperator = (op) => { if (current.value === '') return; if (previous.value !== null) { equal(); } previous.value = current.value; operator.value = op; operatorClicked.value = true; };
const calculate = () => { const prev = parseFloat(previous.value); const curr = parseFloat(current.value); if (isNaN(prev) || isNaN(curr)) return; let result; switch (operator.value) { case '+': result = prev + curr; break; case '-': result = prev - curr; break; case '*': result = prev * curr; break; case '/': result = prev / curr; break; default: return; } return result; };
const equal = async () => { 
  if (operator.value && previous.value !== null && current.value !== '') {
    const result = calculate();
    const calculationString = `${previous.value} ${operator.value} ${current.value} = ${result}`;
    
    // De oude code om de geschiedenis lokaal te tonen
    history.value.unshift(calculationString);
    if (history.value.length > 5) {
      history.value.pop();
    }

    // De nieuwe code om de berekening op te slaan in Supabase
    try {
      const { error } = await supabase
        .from('calculations') // De naam van je tabel
        .insert([
          { calculation_string: calculationString }, // De data die je wilt opslaan
        ])

      if (error) throw error;
      
    } catch (error) {
      console.error('Fout bij het opslaan in Supabase:', error.message);
    }

    // De oude code om de state te resetten
    current.value = String(result);
    previous.value = null;
    operator.value = null;
  }
};

</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=VT323&display=swap');

/* --- Algemene Container Stijl --- */
.main-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column; /* NIEUW: Zet items onder elkaar */
  justify-content: center;
  align-items: center;
  font-family: 'Orbitron', sans-serif;
  color: #e0f2fe;
  position: relative; 
  z-index: 1;
  text-align: center; /* NIEUW: Centreert alle tekst binnen de container */
}

/* --- Fase 1: Splash Scherm Stijlen --- */
.splash-screen {
  font-family: 'VT323', monospace;
  font-size: 24px;
  text-shadow: 0 0 10px #00ffc3;
  color: #00ffc3;
}
.splash-screen p {
  overflow: hidden;
  white-space: nowrap;
  margin: 0 auto;
  letter-spacing: .15em;
  border-right: .15em solid #00ffc3;
  animation: typing 2s steps(40, end), blink-caret .75s step-end infinite;
}
.splash-screen .line1 {
  width: 25ch;
}
.splash-screen .line2 {
  width: 33ch;
  animation-delay: 2s;
  animation-fill-mode: backwards;
}

@keyframes typing {
  from { width: 0 }
  to { width: 100% }
}
@keyframes blink-caret {
  from, to { border-color: transparent }
  50% { border-color: #00ffc3; }
}


/* --- Fase 2: Login Scherm Stijlen --- */
.login-screen {
  background: rgba(15, 23, 42, 0.6);
  padding: 40px;
  border-radius: 20px;
  border: 1px solid rgba(56, 189, 248, 0.3);
  box-shadow: 0 0 40px rgba(56, 189, 248, 0.3);
}
.login-screen h1 {
  font-size: 24px;
  letter-spacing: 4px;
  color: #f9a825;
  text-shadow: 0 0 10px #f9a825;
}
.login-screen input {
  background: rgba(0,0,0,0.5);
  border: 1px solid #f9a825;
  color: #f9a825;
  text-align: center;
  font-family: 'VT323', monospace;
  font-size: 32px;
  margin-top: 20px;
  width: 200px;
  padding: 5px;
  outline: none;
}
.login-screen input.error {
  border-color: #ef4444;
  color: #ef4444;
  animation: shake 0.5s;
}
.error-message {
  color: #ef4444;
  margin-top: 10px;
}
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}


/* --- Fase 3: Laadscherm Stijlen --- */
/* .loading-screen heeft geen extra stijlen nodig, het wordt al gecentreerd door main-container */
.loading-screen h1 {
  font-size: 24px;
  letter-spacing: 4px;
  margin-bottom: 30px;
  color: #00ffc3;
}
.system-check {
  list-style: none;
  padding: 0;
  font-family: 'VT323', monospace;
  font-size: 22px;
}
.system-check li {
  margin-bottom: 10px;
  opacity: 0;
  transform: translateY(10px);
  transition: all 0.5s ease;
}
.system-check li.visible {
  opacity: 1;
  transform: translateY(0);
}
.system-check .status {
  color: #00ffc3;
  margin-right: 15px;
  font-weight: bold;
}

/* --- Fase 4: Holoculator Stijlen --- */
.holoculator-wrapper { 
    display: flex; 
    gap: 40px; 
    align-items: center; /* AANGEPAST: nu ook verticaal gecentreerd */
}
.calculator { width: 360px; background: rgba(15, 23, 42, 0.6); backdrop-filter: blur(10px); border: 1px solid rgba(56, 189, 248, 0.3); border-radius: 20px; overflow: hidden; box-shadow: 0 0 40px rgba(56, 189, 248, 0.3), inset 0 0 10px rgba(56, 189, 248, 0.2); display: grid; grid-template-columns: repeat(4, 1fr); grid-auto-rows: minmax(80px, auto); }
.display { grid-column: 1 / 5; background: rgba(0, 0, 0, 0.3); color: #00ffc3; font-family: 'VT323', monospace; font-size: 64px; padding: 20px; text-align: right; text-shadow: 0 0 10px #00ffc3, 0 0 20px #00ffc3; position: relative; overflow: hidden; border-bottom: 1px solid rgba(56, 189, 248, 0.3); }
.scanline { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: linear-gradient(to bottom, rgba(255, 255, 255, 0), rgba(255, 255, 255, 0.05) 50%, rgba(255, 255, 255, 0)); background-size: 100% 4px; animation: scan 10s linear infinite; }
@keyframes scan { from { background-position-y: 0px; } to { background-position-y: -400px; } }
.btn { background: rgba(56, 189, 248, 0.1); border: none; border-top: 1px solid rgba(56, 189, 248, 0.2); border-left: 1px solid rgba(56, 189, 248, 0.2); color: #e0f2fe; font-family: 'Orbitron', sans-serif; font-size: 24px; display: flex; justify-content: center; align-items: center; cursor: pointer; transition: all 0.2s ease; }
.btn:hover { background: rgba(56, 189, 248, 0.2); color: white; text-shadow: 0 0 5px white; }
.btn:active { background: rgba(56, 189, 248, 0.3); box-shadow: inset 0 0 15px rgba(56, 189, 248, 0.5); transform: translateY(2px); }
.zero { grid-column: 1 / 3; }
.operator { background: rgba(168, 85, 247, 0.2); color: #d8b4fe; }
.operator:hover { background: rgba(168, 85, 247, 0.3); }
.operator:active { box-shadow: inset 0 0 15px rgba(168, 85, 247, 0.5); }
.func { background: rgba(100, 116, 139, 0.2); color: #cbd5e1; }
.func:hover { background: rgba(100, 116, 139, 0.3); }
.func:active { box-shadow: inset 0 0 15px rgba(100, 116, 139, 0.5); }
.history { width: 250px; height: 400px; background: rgba(15, 23, 42, 0.6); backdrop-filter: blur(10px); border: 1px solid rgba(100, 116, 139, 0.3); border-radius: 20px; padding: 15px; color: #94a3b8; font-family: 'VT323', monospace; font-size: 18px; display: flex; flex-direction: column; }
.history-title { color: #e0f2fe; font-family: 'Orbitron', sans-serif; font-size: 16px; text-align: center; margin-bottom: 10px; padding-bottom: 5px; border-bottom: 1px solid rgba(100, 116, 139, 0.3); letter-spacing: 2px; }

/* DE CODE VOOR DE SOEPELE OVERGANG */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease, transform 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
</style>