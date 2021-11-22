<template>
<div class="vuerification">
  <form
    :class="[{ '--error' : isError }, 'vuerification__form']"
    @submit.prevent="vuerificate"
  >
    <fieldset class="vuerification__inputs">
      <input
        v-for="(item, index) in inputModels"
        :key="index"
        :ref="el => { inputElements[index] = el }"
        v-model="inputModels[index]"
        type="number"
        @keydown="inputHandler($event, index)"
        @paste="pasteHandler"
        class="vuerification__input"
      >
    </fieldset>

    <button
      v-if="confirmable"
      type="submit"
      class="vuerification__submit"
    >{{ confirmButton }}</button>
  </form>

  <div
    v-if="isError && errorMessage"
    class="vuerification__error"
  >{{ errorMessage }}</div>
</div>
</template>

<script>
import {
  defineComponent,
  onBeforeUpdate,
  ref,
  reactive,
  watchEffect,
} from 'vue';

export default defineComponent({
  props: {
    length: {
      type: Number,
      required: false,
      default: 4,
    },

    confirmable: {
      type: Boolean,
      required: false,
      default: false,
    },

    confirmButton: {
      type: String,
      required: false,
      default: 'Submit',
    },

    isError: {
      type: Boolean,
      required: true,
    },

    errorMessage: {
      type: String,
      required: false,
      default: '',
    },
  },

  setup(props, { emit }) {
    const inputModels = reactive(Array.from({ length: props.length }, () => null));
    const validKeys = reactive(Array.from({ length: 10 }, (a, b) => {
      const index = b + 1;
      return `Digit${index}`;
    }));
    const inputElements = ref([]);

    validKeys.push('Backspace', 'ControlLeft', 'ControlRight', 'KeyV', 'Enter', 'AltLeft', 'AltRight');

    onBeforeUpdate(() => {
      const arr = [];
      inputElements.value = arr;
    });

    watchEffect(() => {
      if (props.isError) {
        inputElements.value.forEach((el) => {
          /* eslint-disable no-param-reassign */
          el.disabled = false;
        });
      }
    });

    const vuerificate = () => {
      const isFilled = inputModels.every((el) => el);
      const fullCode = Number(inputModels.reduce((acc, next) => {
        /* eslint-disable no-param-reassign */
        acc += next;
        return acc;
      }, ''));

      if (isFilled) {
        inputElements.value.forEach((el) => {
          /* eslint-disable no-param-reassign */
          el.disabled = true;
        });
        emit('vuerificate', fullCode);
      }
    };

    const isKeyValid = (key) => validKeys.includes(key);

    const inputHandler = (e, i) => {
      if (isKeyValid(e.code)) {
        if (['Enter', 'AltLeft', 'AltRight', 'ControlLeft', 'ControlRight'].includes(e.code)) {
          return false;
        }

        if (e.code !== 'Backspace') {
          if (inputModels[i]) {
            inputModels[i] = '';
          }

          if (i < inputModels.length - 1) {
            setTimeout(() => inputElements.value[i + 1].focus(), 0);
          } else if (!props.confirmable) {
            return setTimeout(() => vuerificate(), 0);
          }
        } else if (i > 0) {
          return setTimeout(() => inputElements.value[i - 1].focus(), 0);
        }
      } else {
        return setTimeout(() => {
          inputElements.value[i].value = null;
          inputModels[i] = null;
        }, 0);
      }

      return true;
    };

    const pasteHandler = (e) => {
      const paste = (e.clipboardData || window.clipboardData).getData('text').split('');

      paste.forEach((el, index) => {
        if (!Number.isNaN(el)) {
          inputModels[index] = el;
          inputElements.value[index].value = el;
        }

        setTimeout(() => inputElements.value[index].blur(), 0);
      });

      e.preventDefault();

      if (!props.confirmable) {
        vuerificate();
      }
    };

    return {
      inputModels,
      inputElements,
      inputHandler,
      pasteHandler,
      vuerificate,
    };
  },
});
</script>

<style>
.vuerification {
  font: inherit;
}

.vuerification .vuerification__form {
  text-align: center;
}

.vuerification .vuerification__form.--error .vuerification__input {
  border-color: #f51e38;
}

.vuerification .vuerification__inputs {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 10px 0;
  padding: 0;
  border: 0;
}

.vuerification .vuerification__input {
  flex: none;
  width: 50px;
  height: 50px;
  border-radius: 5px;
  border: 1px solid #cecece;
  text-align: center;
  margin: 0 3px;
  font-size: 20px;
  font-weight: 600;
  appearance: textfield;
  box-shadow: none;
}

.vuerification .vuerification__input::-webkit-outer-spin-button,
.vuerification .vuerification__input::-webkit-inner-spin-button {
  appearance: none;
  margin: 0;
}

.vuerification .vuerification__submit {
  height: 50px;
  width: 218px;
  border-radius: 5px;
  font-size: 18px;
  cursor: pointer;
  border: 1px solid #cecece;
  background-color: #cecece;
}

.vuerification .vuerification__error {
  text-align: center;
  color: #f51e38;
  margin-top: 10px;
  font-size: 18px;
}
</style>
