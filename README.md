<!--HTML-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Calculadora Web</title>
    <link rel="stylesheet" href="calculadoraWebParaPáginaDinâmica.css">
</head>
<body>
    <div id="calculator">
        <div id="display">0</div>
        <div id="buttons">
            <button class="num" id="0">0</button>
            <button class="num" id="1">1</button>
            <button class="num" id="2">2</button>
            <button class="num" id="3">3</button>
            <button class="num" id="4">4</button>
            <button class="num" id="5">5</button>
            <button class="num" id="6">6</button>
            <button class="num" id="7">7</button>
            <button class="num" id="8">8</button>
            <button class="num" id="9">9</button>
            <button class="op" id="add">+</button>
            <button class="op" id="sub">-</button>
            <button class="op" id="mul">*</button>
            <button class="op" id="div">/</button>
            <button id="clear">C</button>
            <button id="equals">=</button>
        </div>
    </div>
    <script src="calculadoraWebParaPáginaDinâmica.js"></script>
</body>
</html>

/*CSS*/
body {
    font-family: Arial, sans-serif;
    text-align: center;
    background-color: #f5f5f5;
}
 
#calculator {
    border: 1px solid #ccc;
    border-radius: 4px;
    width: 300px;
    margin: auto;
    margin-top: 50px;
    background-color: #fff;
}
 
#display {
    font-size: 2em;
    padding: 20px;
    border-bottom: 1px solid #ccc;
    background-color: #f9f9f9;
    color: #333;
}
 
#buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
}
 
button {
    width: 100%;
    padding: 20px;
    font-size: 1.5em;
    border: 1px solid #ccc;
    cursor: pointer;
    transition: background-color 0.3s;
}
 
button:hover {
    background-color: #ddd;
}
 
.op {
    background-color: #f1c40f;
    color: white;
}
 
.op:hover {
    background-color: #f39c12;
}

//Javascript(dinamizando a página//
const calculator = document.getElementById('calculator');
const display = document.getElementById('display');
const buttons = document.getElementById('buttons');

// Histórico persistente
const historyDiv = document.createElement('div');
historyDiv.id = 'history';
historyDiv.style.cssText = `
  font-size: 0.9em; color: #666; padding: 10px; 
  max-height: 100px; overflow-y: auto; text-align: right; 
  background: #f0f0f0; border-bottom: 1px solid #ddd;
`;
calculator.insertBefore(historyDiv, display);

let currentInput = '0';
let operator = null;
let previousInput = null;
let shouldResetDisplay = false;
let history = JSON.parse(localStorage.getItem('calcHistory')) || [];

const updateDisplay = () => {
  display.textContent = currentInput;
};

const updateHistory = () => {
  historyDiv.innerHTML = history.length ? history.join('<br>') : 'Sem histórico';
};

const saveToHistory = (operation, result) => {
  const entry = `${operation} = ${result}`;
  history.unshift(entry);
  history = history.slice(0, 10);
  localStorage.setItem('calcHistory', JSON.stringify(history));
  updateHistory();
};

const clearAll = () => {
  currentInput = '0';
  operator = null;
  previousInput = null;
  shouldResetDisplay = false;
  updateDisplay();
};

const clearHistory = () => {
  history = [];
  localStorage.removeItem('calcHistory');
  updateHistory();
};

const inputNumber = (num) => {
  if (shouldResetDisplay || currentInput === '0' || currentInput === 'Erro') {
    currentInput = num;
    shouldResetDisplay = false;
  } else {
    currentInput += num;
  }
  updateDisplay();
};

const inputOperator = (nextOperator) => {
  const inputValue = parseFloat(currentInput);
  
  if (previousInput === null && !operator) {
    previousInput = inputValue;
  } else if (operator) {
    const result = calculate(previousInput, operator, inputValue);
    if (isNaN(result) || !isFinite(result)) {
      currentInput = 'Erro';
      updateDisplay();
      saveToHistory(`${previousInput} ${getOpSymbol(operator)} ${inputValue}`, 'Erro');
      clearAll();
      return;
    }
    previousInput = result;
    saveToHistory(`${previousInput} ${getOpSymbol(operator)} ${inputValue}`, result);
    updateDisplay();
  }
  
  shouldResetDisplay = true;
  operator = nextOperator;
};

const equals = () => {
  const inputValue = parseFloat(currentInput);
  
  if (operator && previousInput !== null) {
    const result = calculate(previousInput, operator, inputValue);
    
    if (isNaN(result) || !isFinite(result)) {
      currentInput = 'Erro';
      saveToHistory(`${previousInput} ${getOpSymbol(operator)} ${inputValue}`, 'Erro');
    } else {
      currentInput = String(Math.round(result * 100000) / 100000);
      saveToHistory(`${previousInput} ${getOpSymbol(operator)} ${inputValue}`, result);
      previousInput = parseFloat(currentInput);
    }
    operator = null;
    shouldResetDisplay = true;
  }
  updateDisplay();
};

const getOpSymbol = (op) => {
  const ops = { add: '+', sub: '-', mul: '*', div: '/' };
  return ops[op] || op;
};

const calculate = (firstNum, operator, secondNum) => {
  switch (operator) {
    case 'add': return firstNum + secondNum;
    case 'sub': return firstNum - secondNum;
    case 'mul': return firstNum * secondNum;
    case 'div': return secondNum === 0 ? NaN : firstNum / secondNum;
    default: return secondNum;
  }
};

// Event delegation para botões + feedback visual
buttons.addEventListener('click', (e) => {
  const key = e.target;
  const pressedClass = 'pressed';
  
  // Feedback visual dinâmico
  key.classList.add(pressedClass);
  setTimeout(() => key.classList.remove(pressedClass), 150);
  
  if (key.classList.contains('num')) {
    inputNumber(key.textContent);
  } else if (['add', 'sub', 'mul', 'div'].includes(key.id)) {
    inputOperator(key.id);
  } else if (key.id === 'equals') {
    equals();
  } else if (key.id === 'clear') {
    clearAll();
  }
});

// Suporte completo ao teclado
document.addEventListener('keydown', (e) => {
  const key = e.key;
  if (/[0-9]/.test(key)) {
    document.getElementById(key)?.click();
  } else if (key === '+') document.getElementById('add')?.click();
  else if (key === '-') document.getElementById('sub')?.click();
  else if (key === '*') document.getElementById('mul')?.click();
  else if (key === '/' || key === 'Divide') document.getElementById('div')?.click();
  else if (key === 'Enter' || key === '=') document.getElementById('equals')?.click();
  else if (e.key === 'Escape' || key.toLowerCase() === 'c') clearAll();
});

// Inicialização completa
updateDisplay();
updateHistory();
