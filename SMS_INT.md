# SMS service integration.

eskiz.uz kabi providerlar orqali foydalanuvchilarga SMS xabarlar yuborishimiz mumkin.
OTP kodlar, saytdan muhim bildirishnomalar(misol: buyurtma holati haqida), reklamalar kabilar uchun foydalansak bo'ladi.

MCHJ va unga ochilgan account kerak bo'ladi.

- Login qilasiz.
- SMS uchun avvaldan shablon yaratib olasiz.
- Login parol yaratilanadi va ular proyektni .env fayliga qo'yiladi.

Kod misollida:

````
import axios from 'axios'; 

const dotenv = require('dotenv');

dotenv.config();

const SUCCESS = 200;
const PROCESSING = 102;
const FAILED = 400;
const INVALID_NUMBER = 160;
const MESSAGE_IS_EMPTY = 170;
const SMS_NOT_FOUND = 404;
const SMS_SERVICE_NOT_TURNED = 600;

const ESKIZ_EMAIL = process.env.ESKIZ_EMAIL;
const ESKIZ_PASSWORD = process.env.ESKIZ_PASSWORD;

class SendSmsApiWithEskiz {
  private message: string;
  private phone: string;
  private spend: any;
  private email: string;
  private password: string;

  constructor(message: string, phone: string, email: string, password: string) {
    this.message = message;
    this.phone = phone;
    this.spend = null;
    this.email = email;
    this.password = password;
  }

  async send() {
    try {
      const statusCode = this.customValidation();
      if (statusCode !== SUCCESS) {
        return statusCode;
      }

      console.log(`__message`, this.message);
      const result = await this.sendMessage(this.message);
      console.log('__result', result);
      return result === SUCCESS ? result : result;
    } catch (e) {
      console.log('error', e);
      return FAILED;
    }
  }

  customValidation() {
    if (this.phone.toString().length !== 9) {
      return INVALID_NUMBER;
    }
    if (!this.message || this.message.trim().length === 0) {
      return MESSAGE_IS_EMPTY;
    }

    this.message = this.cleanMessage(this.message);
    return SUCCESS;
  }

  async authorization() {
    const uri = 'http://notify.eskiz.uz/api/auth/login';

    const payload = { email: this.email, password: this.password };
    console.log(`authorization__data`);

    try {
      const res = await axios.post(uri, payload, {
        headers: {
          'Content-Type': 'application/json',
          Accept: 'application/json',
        },
      });
      const resJson = res.data;
      console.log(`__resJson`, resJson);
      if (!resJson.data || !resJson.data.token) {
        return FAILED;
      }

      return resJson.data.token;
    } catch (error) {
      console.log('Authorization error:', error.response?.data || error.message);
      return FAILED;
    }
  }

  async sendMessage(message: string) {
    const token = await this.authorization();
    console.log(`__token`, token);
    if (token === FAILED) {
      return FAILED;
    }

    const uri = 'http://notify.eskiz.uz/api/message/sms/send';
    const payload = {
      mobile_phone: '998' + this.phone.toString(),
      message: message,
      from: '4546',
      callback_url: 'http://afbaf9e5a0a6.ngrok.io/sms-api-result/',
    };

    const headers = { Authorization: `Bearer ${token}` };
    console.log(`__headers`, headers);

    try {
      const res = await axios.post(uri, payload, { headers });
      console.log(`__res`, res.status);
      const resJson = res.data;
      console.log(`Eskiz: ${JSON.stringify(resJson)}`);
      return res.status;
    } catch (error) {
      console.log('Send message error:', error.response?.data || error.message);
      return FAILED;
    }
  }

  async getStatus(id: string) {
    const token = await this.authorization();

    const uri = `http://notify.eskiz.uz/api/message/sms/status/${id}`;
    const headers = { Authorization: `Bearer ${token}` };

    try {
      const res = await axios.get(uri, { headers });
      const resJson = res.data;

      if (resJson.status === 'success') {
        if (['DELIVRD', 'TRANSMTD'].includes(resJson.message.status)) {
          return SUCCESS;
        } else if (resJson.message.status === 'EXPIRED') {
          return FAILED;
        } else {
          return PROCESSING;
        }
      } else {
        return FAILED;
      }
    } catch (error) {
      console.log('Get status error:', error.response?.data || error.message);
      return FAILED;
    }
  }

  cleanMessage(message: string) {
    return message.trim();
  }

  async calculationSendSms(message: string) {
    // This method can be used for any pre-send calculations
    return SUCCESS;
  }
}

export async function sendOderOTPMSViaEskizService(phone_number: string, message: string) {
  // Remove the '+' prefix and ensure it's a 9-digit number
  const cleanPhone = phone_number.replace('+', '').replace('998', '');

  const smsService = new SendSmsApiWithEskiz(`najottalim.uz platformasiga kirish uchun tasdiqlash kodingiz: ${message}`, cleanPhone, ESKIZ_EMAIL, ESKIZ_PASSWORD);

  try {
    const result = await smsService.send();
    console.log(`SMS send result: ${result}`);
    return result;
  } catch (e) {
    console.log('SMS sending error:', e);
    return FAILED;
  }
}
````

