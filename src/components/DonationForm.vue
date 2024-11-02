<template>
    <div class="form-container">
        <h2>{{ fundraiserName }} собирает {{ amount }} ₽ на «{{ fundraiserGoal }}»</h2>
        <form @submit.prevent="submitForm">
            <div class="input-group">
                <label>Номер карты</label>
                <input type="text" v-model="cardNumber" v-mask="'#### #### #### ####'" placeholder="Введите номер карты"
                    :class="{ 'error-input': !isCardNumberValid && cardNumber }" />
                <div v-if="!isCardNumberValid && cardNumber" class="error-text">Введите номер карты</div>
            </div>

            <div class="row">
                <div class="input-group">
                    <label>Срок действия</label>
                    <input type="text" v-model="expiryDate" v-mask="'##/##'" placeholder="ММ / ГГ"
                        :class="{ 'error-input': !isExpiryDateValid && expiryDate }" />
                    <div v-if="!isExpiryDateValid && expiryDate" class="error-text">Введите срок</div>
                </div>
                <div class="input-group">
                    <label>CVV</label>
                    <input type="password" v-model="cvv" v-mask="'###'"
                        :class="{ 'error-input': !isCvvValid && cvv }" />
                    <div v-if="!isCvvValid && cvv" class="error-text">Введите CVV</div>
                </div>
            </div>

            <div class="input-group">
                <label>Сумма перевода</label>
                <div class="amount-input">
                    <input type="number" v-model="amount" placeholder="₽"
                        :class="{ 'error-input': !isAmountValid && amount }" min="10" />
                </div>
                <div v-if="!isAmountValid && amount" class="error-text">Введите сумму</div>
            </div>

            <div class="input-group">
                <label>Ваше имя</label>
                <input type="text" v-model="name" placeholder="Введите ваше имя" maxlength="50"
                    :class="{ 'error-input': !isNameValid && name }" />
                <div v-if="!isNameValid && name" class="error-text">Введите имя</div>
            </div>

            <div class="input-group">
                <label>Сообщение получателю</label>
                <textarea v-model="message" placeholder="Сообщение получателю" maxlength="50"></textarea>
            </div>

            <div class="buttons">
                <button type="submit">Перевести</button>
                <button type="button" @click="goBack">Вернуться</button>
            </div>
        </form>
    </div>
</template>

<script>
import { mask } from "vue-the-mask";
import CryptoJS from "crypto-js";

export default {
    directives: { mask },
    data() {
        return {
            fundraiserName: "Иван К.",
            fundraiserGoal: "Экскурсия",
            cardNumber: "",
            expiryDate: "",
            cvv: "",
            amount: 10000,
            name: "",
            message: "Экскурсия",
        };
    },
    computed: {
        isCardNumberValid() {
            return this.validateCardNumber(this.cardNumber.replace(/\s+/g, ""));
        },
        isExpiryDateValid() {
            return /\d{2}\/\d{2}/.test(this.expiryDate);
        },
        isCvvValid() {
            return /\d{3}/.test(this.cvv);
        },
        isAmountValid() {
            return this.amount >= 10;
        },
        isNameValid() {
            return this.name.length > 0;
        },
    },
    methods: {
        validateCardNumber(number) {
            let sum = 0;
            let shouldDouble = false;
            for (let i = number.length - 1; i >= 0; i--) {
                let digit = parseInt(number[i]);
                if (shouldDouble) {
                    digit *= 2;
                    if (digit > 9) digit -= 9;
                }
                sum += digit;
                shouldDouble = !shouldDouble;
            }
            return sum % 10 === 0;
        },
        sha256(data) {
            return CryptoJS.SHA256(data).toString(CryptoJS.enc.Hex);
        },
        submitForm() {
            if (!this.isCardNumberValid || !this.isExpiryDateValid || !this.isCvvValid || !this.isAmountValid || !this.isNameValid) {
                alert("Пожалуйста, исправьте ошибки перед отправкой.");
                return;
            }

            const transactionId = Date.now().toString();
            const amountInCents = this.amount * 100;
            const secretKey = "1234567890";
            const hashSum = this.sha256(`316b2be8-3475-4462-bd57-c7794d4bdb53${transactionId}${amountInCents}${secretKey}`);

            const requestData = {
                api_key: "316b2be8-3475-4462-bd57-c7794d4bdb53",
                transaction: transactionId,
                description: `${this.fundraiserName} собирает на ${this.fundraiserGoal}`,
                amount: this.amount,
                hash_sum: hashSum,
                custom_data: {
                    fundraiserName: this.fundraiserName,
                    fundraiserGoal: this.fundraiserGoal,
                },
            };

            console.log("Данные для отправки:", requestData);
            alert("Данные успешно отправлены");
        },
        goBack() {
            console.log("Возвращаемся на предыдущую страницу");
        },
    },
};
</script>

<style>
.form-container {
    padding: 32px;
    border-radius: 24px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    background-color: #ffffff;
}

.form-container h2 {
    font-size: 23px;
    font-weight: 600;
    color: #333333;
    margin-bottom: 24px;
    text-align: left;
}

.form-container label {
    font-size: 16px;
    color: rgba(34, 37, 43, 0.7);
    margin-bottom: 8px;
    display: block;
    text-align: left;
}

.form-container .input-group {
    margin-bottom: 16px;
}

.form-container input,
.form-container textarea {
    width: 100%;
    max-width: 536px;
    height: 52px;
    padding: 10px;
    border: 1px solid #d1d1d1;
    border-radius: 12px;
    font-size: 16px;
    color: #333;
    box-sizing: border-box;
    transition: border-color 0.2s;
}

.form-container textarea {
    resize: none;
    line-height: 30px;
    /* Для вертикального центрирования текста */
}

.form-container .row {
    display: flex;
    gap: 36px;
}

.form-container .row input:first-child {
    max-width: 240px;
}

.form-container .row input:last-child {
    max-width: 260px;
}

.form-container input::placeholder,
.form-container textarea::placeholder {
    color: #a8a8a8;
}

.form-container input:focus,
.form-container textarea:focus {
    border-color: #3b82f6;
    outline: none;
}

.error-input {
    border-color: #C24100 !important;
}

.error-text {
    font-size: 16px;
    color: #C24100;
    margin-top: 4px;
}

.buttons {
    display: flex;
    gap: 10px;
    margin-top: 16px;
}

.buttons button {
    height: 52px;
    padding: 0 16px;
    border: none;
    border-radius: 12px;
    font-size: 16px;
    cursor: pointer;
    font-weight: 500;
}

.buttons button[type="submit"] {
    width: 131px;
    background-color: #3b82f6;
    color: white;
}

.buttons button[type="button"] {
    width: 129px;
    background-color: #e0e0e0;
    color: #333;
}

.buttons button[type="submit"]:hover {
    background-color: #2563eb;
}

.buttons button[type="button"]:hover {
    background-color: #c0c0c0;
}
</style>
