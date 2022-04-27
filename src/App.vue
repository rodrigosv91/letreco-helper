<template>
    <div class="app-container modal-vue">

        <!-- header -->
        <div class="position-relative mt-3 mb-4 d-flex align-items-center justify-content-around">
          <!-- app name -->
          <h1 class="text-center mb-0 app-name"><NomeApp /></h1>
          <!-- button show modal -->
          <button class="help-button rounded position-absolute align-items-center justify-content-center" 
            @click="showModal = true">
            <svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 16 16" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><path d="M3 4.075a.423.423 0 0 0 .43.44H4.9c.247 0 .442-.2.475-.445.159-1.17.962-2.022 2.393-2.022 1.222 0 2.342.611 2.342 2.082 0 1.132-.668 1.652-1.72 2.444-1.2.872-2.15 1.89-2.082 3.542l.005.386c.003.244.202.44.446.44h1.445c.247 0 .446-.2.446-.446v-.188c0-1.278.487-1.652 1.8-2.647 1.086-.826 2.217-1.743 2.217-3.667C12.667 1.301 10.393 0 7.903 0 5.645 0 3.17 1.053 3.001 4.075zm2.776 10.273c0 .95.758 1.652 1.8 1.652 1.085 0 1.832-.702 1.832-1.652 0-.985-.747-1.675-1.833-1.675-1.04 0-1.799.69-1.799 1.675z"></path></svg>
          </button>          
        </div> 
        
        <!-- app body -->
        <div class="mt-2">
          <div class="mb-4">

            <!-- input -->
            <div class="d-flex justify-content-center align-items-center mb-4 ml-1">           
              <input v-for="item in letras_input" :key="item.index"
                type="text"  maxlength="1" 
                class="letter-wrapper rounded d-flex justify-content-center align-items-center mr-2"
                v-model="letras_input[item.index].value"
                @keypress="mudarLetra($event, item.index)"
                v-bind:class="[letras_input[item.index].state]"
                @dblclick="toggleClassInput(item.index)"
                @keydown="toggleClassThroughArrowKey($event, item.index)"
              >                                
            </div>

            <!-- buttons -->
            <div class="d-flex justify-content-center mb-4">
                <button 
                  class="mr-2 rounded"
                  @click="resetar()" >
                  <span>Resetar</span>
                </button>
                <button
                  class="rounded"
                  @click="procurar()"
                  ><span>Procurar</span>
                </button>
            </div>

            <!-- modal -->
            <div class="d-flex justify-content-center align-items-center ">  
              <div class="modal_box rounded content" v-if="showModal">
                <button class="close" @click="showModal = false">X</button> 
                <ModalContent />
              </div>
            </div>
           
            <hr class="divider">  

            <!-- keyboard buttons -->
            <div class="px-lg-5 px-2">
              <h5 class="text-center m-4">Letras Incorretas</h5>

              <!-- keyboard lines -->
              <div  v-for="linha in listaDeLetrasTeclado" :key="linha.id_linha"
                class="d-flex justify-content-center mb-2"
              >
                <!-- keyboard buttons in line -->
                <button v-for="tecla in linha" :key="tecla.index"
                    class="keyboard-button rounded me-2 letter-button mr-2"
                    type="button" 
                    v-bind:class="[listaDeLetras[tecla.index].state]"
                    @click="toggleClassKeyboardButton(tecla.index)"
                >
                  <span>{{ tecla.value }}</span>
                </button> 
              </div>
            </div>

          </div> 
        </div>
     
        <hr class="divider">      
                       
        <!-- result status-->
        <div class="m-4 word">            
          <div v-if="VerificaSeProcuraInvalida">           
                <h5>Sem Resultados</h5>  
                <p class="m-4">Letra procurada não pode ser igual a letra marcada como incorreta.</p>  
          </div> 
          <div v-else-if="!VerificaSeProcuraInvalida && !listaDePalavrasEncontradas.length  && verificaSeHaPesquisa">           
                <h5>Sem Resultados</h5>  
                <p class="m-4">Não há resultados para esta combinação.</p>  
          </div> 
          <div v-else-if="listaDePalavrasEncontradas.length > 0 && listaDePalavrasEncontradas.length == 1" >
                <h5>Palavra Encontrada</h5>
          </div>
          <div v-else-if="listaDePalavrasEncontradas.length > 1" >
                <h5>Palavras Encontradas</h5>
          </div>
        </div>  

        <!-- results -->   
        <div class="found-words-box d-flex justify-content-center overflow-auto word" > 
          <div class="container">   
              <div class="row justify-content-center" v-for="i in Math.ceil(listaDePalavrasEncontradas.length / 5) " :key="i.id">
                <p  class="col mb-1" v-for="palavra in listaDePalavrasEncontradas.slice((i - 1 ) * 5, i * 5)" :key="palavra.id">
                  <span>{{palavra}}</span>
                </p>           
              </div>        
          </div>
        </div> 
        
        <!-- modal overlay -->
        <div class="overlay" v-if="showModal" @click="showModal = false"></div>
    </div>
</template>

<script>
import NomeApp from './NomeApp'
import ModalContent from './ModalContent'

export default {
  name: 'App', 
  components: {
    NomeApp,
    ModalContent
  },
  data (){
    return{
      word_size: 5,
      letras_input:  [],    // array criado dinamicamente
      listaDePalavras: [],
      listaDePalavrasEncontradas: [],
      VerificaSeProcuraInvalida: false, 
      verificaSeHaPesquisa: false,

      showModal: false ,

      alfabeto: 'QWERTYUIOPASDFGHJKLZXCVBNM'.split(''),
      listaDeLetras: [],
      listaDeLetrasTeclado: [],  // array 'listaDeLetras' adaptado para formato de teclado   
      letras_input_estado_inicial:[],
      listaDeLetras_estado_inicial: []
    }
  },
  created() {
      fetch(`https://rodrigosv91.github.io/Data/words.json`)  // busca todas palavras possiveis
        .then(res => {
          return res.json();
       }).then(this.setWordList),
       
      this.cria_lista_de_inputs(),    // cria array que gerara input para entrada de letras 
      this.cria_lista_de_letras(),    // cria array de letras com 'state' e 'index'
      this.cria_lista_de_letras_teclado()  // cria array que gerara teclado apartir de letras
  },
  methods:{
    setWordList(wordList){
      this.listaDePalavras = wordList;
    },   
    cria_lista_de_inputs(){
        var array_letras_input = []; 
        for(let i = 0; i < this.word_size; i++){
          array_letras_input.push( {'index': i, 'value': "", 'state': "right"});          
        }
        this.letras_input = array_letras_input;
        return array_letras_input;
    },
    cria_lista_de_letras(){
      var array_letras = []; 
      for(let i = 0; i < this.alfabeto.length; i++){
          array_letras.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});          
      }
      this.listaDeLetras = array_letras;
      return array_letras;
    },
    cria_lista_de_letras_teclado(){
      var linha1 = []; 
      var linha2 = []; 
      var linha3 = []; 

      for(let i = 0; i < this.alfabeto.length; i++){
          if(i<10){
            linha1.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});   
          }else
          if(i>=10 && i<19 ){
            linha2.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});   
          }else
          if(i>=19){
            linha3.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});   
          }      
      }
      this.listaDeLetrasTeclado.push(linha1);
      this.listaDeLetrasTeclado.push(linha2);
      this.listaDeLetrasTeclado.push(linha3);
    },
    //muda estado de letra em teclado
    toggleClassKeyboardButton(index){
      /*
      if(this.listaDeLetras[index].state == "wrong"){
        this.listaDeLetras[index].state = "none";
      }else if(this.listaDeLetras[index].state == "none"){
        this.listaDeLetras[index].state = "wrong";
      } */
      this.listaDeLetras[index].state = this.listaDeLetras[index].state == "none" ? "wrong" : "none";
    },
    //muda estado de letra em input
    toggleClassInput(n){
      /*
      if(this.letras_input[n].state == "right"){
        this.letras_input[n].state = "displaced";
      }else if(this.letras_input[n].state == "displaced"){
        this.letras_input[n].state = "right";
      } */
      this.letras_input[n].state = this.letras_input[n].state == "right" ? "displaced" : "right";
    },
    //muda estado de letra em input via Key Arrow
    toggleClassThroughArrowKey(event, InputIndex){
      if(event.key == "ArrowUp" || event.key == "ArrowDown"){
        this.toggleClassInput(InputIndex);
        this.procurar();
      }
      else
      if(event.key == "ArrowRight"){
        event.target.nextElementSibling.focus();
        this.procurar();
      }
      else
      if(event.key == "ArrowLeft"){
        event.target.previousElementSibling.focus();
        this.procurar();
      }
    },
    //muda valor de letra no input
    mudarLetra(e, n){     
      if(e.key == "Enter"){ 
        this.procurar();
      } 

      //muda valor da letra em Data
      let ltr = e.key;

      function exists(arr, search) {
          return arr.some(palavra => palavra.includes(search)); // metodo "some" verifica multidimensional array
      }                                                         // https://stackoverflow.com/questions/52103644/includes-in-multidimensional-array
      if (ltr.length === 1 && exists(this.alfabeto, ltr.toUpperCase() ) ) {  
        this.letras_input[n].value = ltr;
      }else {
        e.preventDefault();
      }
    },
    //procura palavras que s encaixem nos parametros
    procurar(){
      this.listaDePalavrasEncontradas = [];

      var newArray = [];
      var existeLetraBusca = false;
      var existeLetraErrada = false;
      this.VerificaSeProcuraInvalida = false;

      //verifica se existe letra a ser buscada em input
      for (var i = 0; i < this.letras_input.length; ++i) {
        if(this.letras_input[i].value != ''){
          existeLetraBusca = true;
          break;
        }
      }

      //verifica se existe alguma letra errada a ser filtrada das palavras
      let letrasErradas = this.listaDeLetras.filter(function checkWrong(obj) {
        return obj.state == 'wrong';
      });
      existeLetraErrada = letrasErradas.length > 0 ? true : false;

      //carrega lista de palavras se existe alguma letra a ser buscada (input) ou filtrada (letra errada)
      if(existeLetraBusca || existeLetraErrada){
        this.listaDePalavrasEncontradas = this.listaDePalavras;
      }
      
      //verifica se a letra existe na posição "N" da palavra 
      this.letras_input.forEach( item => {
        if(item.value != '' && item.state == 'right'){
          newArray = [];
          this.listaDePalavrasEncontradas.forEach(palavraTestada => {
              if(palavraTestada[item.index] == item.value.toUpperCase())
                newArray.push(palavraTestada);
          });
          this.listaDePalavrasEncontradas = newArray;
        }
      });    
      
      //verifica se existe a letra na palavra        
      this.letras_input.forEach( item => {
        if(item.value != '' && item.state == 'displaced'){
        newArray = [];
        this.listaDePalavrasEncontradas.forEach(palavra => {
            if(palavra.includes(item.value.toUpperCase()) && palavra[item.index] != item.value.toUpperCase() ){
              newArray.push(palavra);
            }
        });
        this.listaDePalavrasEncontradas = newArray;
        }
      });

      //exclue da lista palavras que contem alguma das letras excluidas (wrong)        
      if(existeLetraErrada){
        letrasErradas.forEach(letra_errada => {
          newArray = []; 
          this.listaDePalavrasEncontradas.forEach(palavra => {
            if(!palavra.includes(letra_errada.value)){
              newArray.push(palavra);
            }
          });
          this.listaDePalavrasEncontradas = newArray;
        });      
      }

      //verifica se uma letra buscada foi marcada como inexistente 
      this.letras_input.forEach(letra_input => {
          letrasErradas.forEach(letra_errada => {
            if(letra_input.value.toUpperCase() == letra_errada.value.toUpperCase()){
                this.VerificaSeProcuraInvalida = true;
            }
          });    
      });

      //Houve uma tentativa de pesquisa
      this.verificaSeHaPesquisa = true  
      
    },
    //reseta parametros
    resetar(){
      this.letras_input = this.cria_lista_de_inputs();
      this.listaDeLetras = this.cria_lista_de_letras();
      this.listaDePalavrasEncontradas = [];
      this.VerificaSeProcuraInvalida = false;
      this.verificaSeHaPesquisa = false;
    },
    
  }
}
</script>

<style>
/* latin */
@font-face {
  font-family: 'Acme';
  font-style: normal;
  font-weight: 400;
  src: url(https://fonts.gstatic.com/s/acme/v18/RrQfboBx-C5_XxrBbg.woff2) format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
/* latin */
@font-face {
  font-family: 'Signika Negative';
  font-style: normal;
  font-weight: 400;
  src: url(https://fonts.gstatic.com/s/signikanegative/v18/E21x_cfngu7HiRpPX3ZpNE4kY5zKSPmJXkF0VDD2RAqnS43rvdk.woff2) format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
body {
 background-color:#f4f3f1;
 font-family:Signika Negative,sans-serif
}
.app-container {
  max-width: 500px;
  margin-left: auto;
  margin-right: auto; 
}
.app-name {
  font-family:Acme,sans-serif;
  font-size: 270%;
}
.letter-wrapper {
  border: 2px solid #252525;
  width: 48px;
  height: 40px;

  font-size: 20px;
  font-weight: bolder;
}
input { 
    text-align: center; 
    text-transform: uppercase;
    /*background-color: rgb(81, 179, 110);   */
}
.right{
  background-color: #51b36e;
  color: #f4f3f1;
}
.displaced { 
  background-color: #c79c2e;
  color: #f4f3f1 !important;
}
.wrong {
  background-color: #943e3c;
  color: #f4f3f1;
}
.keyboard-button {
  border: solid 1px #252525;
  font-size: 16px;
}
.letter-button {
  width: 8%;
  height: 40px;
}
.found-words-box{
  max-width: 450px;
  max-height: 250px;
  margin-left: auto;
  margin-right: auto;
}
.col{
  max-width: 20%;
}
.word{
  text-align: center !important;
}
.text-center {
    text-align: center !important;
}
.divider {
  flex-grow: 1;
  border-bottom: 1px solid black;
  margin: 5px
}
hr .divider{
  flex-grow: 1;
  border-bottom: 1px solid black;
  margin: 5px
}
.letter-green {
  color: #51b36e;
}
.letter-red {
  color: #943e3c;
}
.letter-yellow {
  color: #c79c2e;
}
.header-button {
  position: absolute;
  background-color: #f4f3f1;
  border: solid 1px #252525;

  top: 0px;
  right: 0px;
  width: 24px;
  height: 20px;

  font-size: 150%;
  padding: 5px;
}
.help-button {
  position: absolute;
  top: 1%;
  right: 1%;
}
.action-button {
    padding: 5px;
    width: 20%;
}
.modal-vue .overlay {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, .5); /* */
}
.modal-vue .modal_box {
  position: absolute;
  
  z-index: 9999;
  margin: 0 auto;
  padding: 20px 30px;
  background-color: #f4f3f1;
  border: 1px solid #252525;
}
.modal-vue .close{
  position: absolute;
  top: 10px;
  right: 10px;
}
.content {
    max-height: 78vh;
    overflow: auto;
}
</style>
