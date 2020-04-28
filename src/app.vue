<template>
  <div id="app">    
    <messageSystem v-bind:messagesList="messagesList"></messageSystem>
    <searchString v-model="searchText"></searchString>
    <queryForm v-model="queryObject"></queryForm>
    <input type="button" @click="showNewLotForm()" value=" Додати лот" />
    <lotTable v-bind:lotList="filtredLots" @remove="removeLot" @update="showUpdateLotForm"></lotTable>
    <lotForm v-model="lot" v-bind:visible="formVisible" @input="formInput"></lotForm>
  </div>
</template>

<script>
import Vue from "vue";
import lotTable from "./components/lotTable";
import lotForm from "./components/lotForm";
import searchString from "./components/searchString";
import queryForm from "./components/queryForm";
import messageSystem from "./components/messageSystem";
import axios from "axios";

export default {
  //назва компоненту, співпадає із назвою файла  
  name: "app",
  //компоненти які використовуються в додатку
  components: {
    lotTable,
    lotForm,
    searchString,
    queryForm,
    messageSystem
  },
  //дані компонента на початку
  data() {
    return {
      //чписок всіх лотів
      lots: [],
      //обєкт 1 порожнього лота
      lot: {
        title: "",
        startPrice:0,
        endPrice:0,
        startData:"1997-01-10T22:00:00.000Z",
        endData:"1997-01-10T22:00:00.000Z"
      },
      //список повідомлень
      messagesList: [],
      //видимість форми
      formVisible: false,
      //індекс вибраного лота
      selectedIndex: -1,
      //текст пошуку
      searchText: "",
      //об'єкт з параметрами пошуку 
      queryObject: {
        minPages: null,
        maxPages: null,
        maxPrice: null
      }
    };
  },
  // значення які необхідно обчислити.  
  computed: {
    //відфільтрований список лотів
    filtredLots() {
      let result = this.lots;
      if (this.searchText)
        result = result.filter(lot =>
          lot.title.toLowerCase().includes(this.searchText.toLowerCase())
        );
      console.log(result);
      if (this.queryObject.data)
        result = result.filter(lot => ((lot.startData <= this.queryObject.Data)&&(lot.maxData>=this.queryObject.Data)));
      console.log(result);
      
      return result;
    }
  },
  //коли додаток створено то треба завантажити список лотів із сервера 
  mounted() {
    this.getLots().then(lots => {
      this.lots = lots;
    });
  },
  //методи 
  methods: {
    // асинхронний хук по завантаження всіх лотів
    async getLots() {
      try {
        //чекаємо відповідь на запит GET  http://localhost:5000/lot
        let resp = await axios.get("http://localhost:5000/lot");
        //якщо все добре то повертаємо данні із відповіді на запит 
        return resp.data;
      } catch (e) {
        //якщо сталась помилка (в тому числі і сервер повернув 500)
        console.log(e);
        //додати повідомлення 
        this.messagesList.push(e);
      }
    },
    // асинхронний хук по додаванню 1 лота
    async postLot(lot) {
      try {
        let resp = await axios.post("http://localhost:5000/lot", lot);
        return resp.data;
      } catch (e) {
        console.log(e);
        this.messagesList.push(e);
      }
    },
    // асинхронний хук по вилученню 1 лота
    async deleteLot(lot) {
      try {
        let resp = await axios.delete(`http://localhost:5000/lot/${lot._id}`);
        return resp.data;
      } catch (e) {
        console.log(e);
        this.messagesList.push(e);
      }
    },
    // асинхронний хук по оновленню інформації про лот
    async patchLot(lot) {
      try {
        let resp = await axios.patch(
          `http://localhost:5000/lot/${lot._id}`,
          lot
        );
        return resp.data;
      } catch (e) {
        console.log(e);
        this.messagesList.push(e);
      }
    },
    // показуємо форму для нового лота  
    showNewLotForm() {
      this.lot = Object.assign(this.lot, {
        title: "",
        authors: [],
        isbn: "",
        published: "1997-01-10T22:00:00.000Z",
        pages: 0,
        price: 0
      });
      this.formAction = this.addNewLot;
      this.formVisible = true;
    },
    //показуємо форму для оновлення лота
    showUpdateLotForm(index) {
      this.selectedIndex = index;
      Object.assign(this.lot, this.filtredLots[index]);
      this.formAction = this.updateLot;
      this.formVisible = true;
    },
    //додавання нового лота
    addNewLot() {
      //спочатку робимо хук 
      this.postLot(this.lot).
      //потім додаємо лот в список якщо хук пройшов вдало
      then(lot => this.lots.push(lot));
    },
    //вилучення лота
    removeLot(index) {
      this.deleteLot(this.filtredLots[index]).
      then(() => {
        this.filtredLots.splice(index, 1);
      });
    },
    //оновити лот
    updateLot() {
      let i = this.selectedIndex;
      this.patchLot(this.lot).then(lot => {
        Object.assign(this.filtredLots[i], lot);
      });
      this.selectedIndex = -1;
    },
    //показати форму редагування або оновлення лота
    formInput() {
      this.formAction();
      this.formVisible = false;
    },
    formAction: function() {}
  }
};
</script>

<style scoped>
#app {
  margin: 0;
  padding: 0;
}
</style>