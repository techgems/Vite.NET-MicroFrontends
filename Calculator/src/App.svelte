<script>
  import { onMount, afterUpdate } from "svelte";

  import Button from "./Button.svelte";

  const operatorMap = ["+", "-", "*", "/", "="];
  const numericalMap = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "."];

  let equalsPressed = false;
  let expression = "";
  let expressionEquals = "";
  let value = "0";

  let screen;

  afterUpdate(() => {
    handleScreenScroll();
  });

  function handleScreenScroll() {
    if (screen.scrollWidth > 550) {
      screen.style.overflowX = "scroll";
      screen.scrollTo(screen.scrollWidth, 0);
    } else {
      screen.style.overflowX = "auto";
    }
  }

  function isValidExpression(exp) {
    return /^(?:\d+[+\-*\/]|\d+\.\d+[+\-*\/]|\-\d+[+\-*\/]|\.\d+[+\-*\/])+(?:(?<=[\+\*\/])\-\d+|\.\d+|\d+|\d+\.\d+)$/.test(
      exp
    );
  }

  function onNumericalKeyPress(v) {
    value === "0" ? (value = v) : (value += v);
    expression += v;
  }

  function clearEqualsPressed() {
    if (equalsPressed) {
      equalsPressed = false;
    }
  }

  function onBackKeyPress() {
    clearEqualsPressed();

    if (expression.length > 1 && isValidOperator) {
      //remove last char
      expression = expression.substring(0, expression.length - 1);

      //Another check to make sure the expression stll has multiple chars after initial removal
      if (expression.length > 1) {
        let opsInExpression = expression.match(/\+|\-|\*|\//g);
        let valueFromExp;

        //Check that expression only has one complete number and no other  operators
        if (
          !!opsInExpression &&
          (opsInExpression.length === 1 && expression[0] === "-")
        ) {
          value = expression;
        } else {
          value = expression.match(/(\-\d+$|\d+$)/g)[0];
        }
      } else {
        value = expression;
      }
    } else {
      if (value.length > 1) {
        value = value.substring(0, value.length - 1);
      } else {
        value = "0";
      }
      expression = expression.substring(0, expression.length - 1);
    }
  }

  function evaluateExpression() {
    //avoiding eval by using function constructor
    let ev = new Function(`return ${expression}`);
    expressionEquals = ev();
  }

  function onEqualsPress() {
    if (isValidExpression(expression)) {
      equalsPressed = true;
      evaluateExpression();
    }
  }

  function onClearKeyPress() {
    value = "0";
    expression = "";
    expressionEquals = "";
    clearEqualsPressed();
  }

  function handleKeyPresses(event) {
    if (operatorMap.includes(event.key)) {
      onOperatorKeyPress(event.key);
    } else if (numericalMap.includes(event.key)) {
      onNumericalKeyPress(event.key);
    } else if (event.key === "Backspace") {
      onBackKeyPress();
    } else if (event.key === "Escape") {
      onClearKeyPress();
    } else if (event.key === "Enter") {
      onEqualsPress();
    }
  }

  function onOperatorKeyPress(operator) {
    clearEqualsPressed();

    if (value === "0" && operator === "-") {
      value = operator;
      expression += operator;
    } else if (value !== "0" && !lastCharIsOperator) {
      expression += operator;
      value = "0";
    }
  }

  $: isValidOperator = /(?<![\*\/\+\-])[\*\/\+\-]$/.test(expression);

  $: lastCharIsOperator = operatorMap.includes(
    expression[expression.length - 1]
  );

  $: roundedEquals = /\./.test(expressionEquals)
    ? Number(expressionEquals).toFixed(2)
    : expressionEquals;

  $: isValidExpression(expression)
    ? evaluateExpression()
    : (expressionEquals = "");

  $: expressionWithEntities = expression.replace(
    /(?:\*)|(?:\/)|(?:\+)|(?:\-)/g,
    match => {
      if (match === "*") {
        return " &times; ";
      } else if (match === "/") {
        return " &divide; ";
      } else if (match === "+") {
        return " &plus; ";
      } else if (match === "-") {
        return " &minus; ";
      } else {
        return match;
      }
    }
  );
</script>

<style>
  h4 {
    color: #808090;
  }

  .container {
    display: flex;
    flex-flow: row nowrap;
    width: 100%;
    height: 100%;
    background: #ffddab;
    justify-content: center;
    align-items: center;
  }

  main {
    display: flex;
    flex-direction: column;
    width: 550px;
    height: 700px;
    margin-top: -40px;
    padding: 10px;
    background: #cdd1d4;
    border: 2px solid #808090;
    box-shadow: 1px 2px 9px 2px #808090;
  }

  .screen {
    flex: 0 0 auto;
    display: grid;
    grid-template-columns: 1fr auto;
    grid-template-rows: 30px auto;
    width: 550px;
    height: 85px;
    border: 2px solid #808090;
    border-radius: 5px;
    overflow-y: hidden;
    overflow-x: scroll;
    background: #ffffff;
  }

  ::-webkit-scrollbar {
    height: 10px;
  }

  ::-webkit-scrollbar-track {
    background: #f1f1f1;
  }

  ::-webkit-scrollbar-thumb {
    background: #a0a3a7;
    border-radius: 10px;
  }

  /* Handle on hover */
  ::-webkit-scrollbar-thumb:hover {
    background: #555;
  }

  .exp-text {
    text-align: end;
  }

  .val {
    text-align: end;
    grid-column: 1 / -1;
  }

  .button-container {
    margin-top: 10px;
    flex: 1 1 auto;
    display: grid;
    grid-gap: 5px;
    grid-template-columns: 3fr 1fr;
  }

  .action-buttons {
    display: grid;
    grid-gap: 5px;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(5, 1fr);
  }

  .arithmetic-buttons {
    display: grid;
    grid-gap: 5px;
    grid-template-rows: repeat(5, 1fr);
  }

  @media screen and (max-height: 780px) {
    main {
      margin-top: 0px;
    }
  }
</style>

<svelte:window on:keydown={handleKeyPresses} />

<div class="container">
  <main>

    <div bind:this={screen} class="screen">

      <div class="exp-text">
        <h4>
          {@html expressionWithEntities}
        </h4>
      </div>
      <div class="exp-sum">
        {#if equalsPressed}
          <h4>=</h4>
        {:else if !!expressionEquals}
          <h4>{`= ${roundedEquals}`}</h4>
        {/if}
      </div>
      <div class="val">
        <h1>{equalsPressed ? expressionEquals : value}</h1>
      </div>

    </div>

    <div class="button-container">

      <div class="action-buttons">
        <Button onClick={() => onNumericalKeyPress('1')}>1</Button>
        <Button onClick={() => onNumericalKeyPress('2')}>2</Button>
        <Button onClick={() => onNumericalKeyPress('3')}>3</Button>
        <Button onClick={() => onNumericalKeyPress('4')}>4</Button>
        <Button onClick={() => onNumericalKeyPress('5')}>5</Button>
        <Button onClick={() => onNumericalKeyPress('6')}>6</Button>
        <Button onClick={() => onNumericalKeyPress('7')}>7</Button>
        <Button onClick={() => onNumericalKeyPress('8')}>8</Button>
        <Button onClick={() => onNumericalKeyPress('9')}>9</Button>
        <Button onClick={() => onNumericalKeyPress('.')}>.</Button>
        <Button onClick={() => onNumericalKeyPress('0')}>0</Button>
        <Button type="action" onClick={() => onBackKeyPress()}>&crarr;</Button>
        <Button type="action" spanRow onClick={() => onClearKeyPress()}>
          CLEAR
        </Button>
      </div>
      <div class="arithmetic-buttons">
        <Button type="operator" onClick={() => onOperatorKeyPress('+')}>
          &plus;
        </Button>
        <Button type="operator" onClick={() => onOperatorKeyPress('-')}>
          &minus;
        </Button>
        <Button type="operator" onClick={() => onOperatorKeyPress('*')}>
          &times;
        </Button>
        <Button type="operator" onClick={() => onOperatorKeyPress('/')}>
          &divide;
        </Button>
        <Button type="operator" onClick={() => onEqualsPress()}>=</Button>
      </div>

    </div>
  </main>
</div>
