# zxc
// SPDX-License-Identifier: MIT
pragma solidity >0.8.0;
contract con {
  struct polz { //структура ппользователей
    uint invest_privat; //токены
    uint invest_pub;
    bytes32 pwhash; //пароль
    uint role; // 1-инвестор, 2-овнер, 3-privat Провайдер, 4-pub Провайдер
  }
  struct privtoken { //структура приватного токена
    address investor; //адресс инвесторов 
    uint value; //цена
    bool finished; //завершение
  }
  struct pubtoken { //структура публичного токена
    address investor; //адресс инвесторов
    uint value; //цена
    bool finished; //завершение
  }
  struct obmen { //структрута обмена
    address adr_from; //адрес отправителя
    address adr_to; //адресс получателя
    uint value_pub; //цена публичных паблик токенов
    uint value_privat; //цена приватных паблик токенов
    bool finished; //завершение
  }
  mapping (address => polz) public Polzovateli; //соответствие для внесения пользователя
  address[] public polz_list; //массив пользователей
  mapping (uint => privtoken) privtokens; 
  uint public privtoken_amount; //массив операций
  mapping (uint => pubtoken) pubtokens;
  uint public pubtoken_amount; //массив опираций
  constructor () {
    Polzovateli[0x5B38Da6a701c568545dCfcB03FcB875f56beddC4] = 
polz(1000,1000,0x64e604787cbf194841e7b68d7cd28786f6c9a0a3ab9f8b0a0e87cb4387ab0107,1); //инвесторы
    Polzovateli[0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2] = 
polz(1000,1000,0x64e604787cbf194841e7b68d7cd28786f6c9a0a3ab9f8b0a0e87cb4387ab0107,1);
    Polzovateli[0x617F2E2fD72FD9D5503197092aC168c91465E7f2] = 
polz(100000,100000,0x64e604787cbf194841e7b68d7cd28786f6c9a0a3ab9f8b0a0e87cb4387ab0107,3); //privat Провайдер
    Polzovateli[0x17F6AD8Ef982297579C203069C1DbfFE4348c372] = 
polz(100000,100000,0x64e604787cbf194841e7b68d7cd28786f6c9a0a3ab9f8b0a0e87cb4387ab0107, 4); //pub Провайдер
    Polzovateli[0x03C6FcED478cBbC9a4FAB34eF9f40767739D1Ff7] = 
polz(10000000,10000000,0x64e604787cbf194841e7b68d7cd28786f6c9a0a3ab9f8b0a0e87cb4387ab0107,2); //овнер
    polz_list.push(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4); //инвесторы
    polz_list.push(0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2);
    polz_list.push(0x617F2E2fD72FD9D5503197092aC168c91465E7f2); //privat Провайдер
    polz_list.push(0x17F6AD8Ef982297579C203069C1DbfFE4348c372); //pub Провайдер
    polz_list.push(0x03C6FcED478cBbC9a4FAB34eF9f40767739D1Ff7); //овнер
  }
  function create_investor (bytes32 pwhash ) public { //функция регестрации инвестора
    require(Polzovateli[msg.sender].pwhash == 0,"pol`zovatel ect`");
    Polzovateli[msg.sender] = polz(0,0,pwhash,1);
    polz_list.push(msg.sender);
  }
  function create_private (uint value) public { //создание приватного перевода
    require(value > 0,"nevernya cymma");
  privtokens[privtoken_amount] = privtoken(msg.sender,value,false);
  privtoken_amount += 1;
  Polzovateli[0x03C6FcED478cBbC9a4FAB34eF9f40767739D1Ff7].invest_privat -= value;
  }
  function stop_privat (uint id) public { //остановка приватного перевода
    require(Polzovateli[msg.sender].role == 3,"ne provaider");
    require(privtokens[id].finished == false,"zaverchen");
    privtokens[id].finished = true;
    Polzovateli[0x03C6FcED478cBbC9a4FAB34eF9f40767739D1Ff7].invest_privat += privtokens[id].value;
  }
  function get_private (uint id) public { //подтверждение
    require(Polzovateli[msg.sender].role == 3,"ne provaider");
    require(privtokens[id].finished == false,"zaverchen");
    privtokens[id].finished = true;
    address investor = privtokens[id].investor;
    Polzovateli[investor].invest_privat += privtokens[id].value ;
  }
  function create_pub (uint value) public { //создание публичного перевода
    require(value > 0,"nevernya cymma");
  pubtokens[privtoken_amount] = pubtoken(msg.sender,value,false);
  pubtoken_amount += 1;
  Polzovateli[0x03C6FcED478cBbC9a4FAB34eF9f40767739D1Ff7].invest_pub -= value;
  }
  function stop_pub (uint id) public { //остановка публичного перевода
    require(Polzovateli[msg.sender].role == 4,"ne provaider");
    require(privtokens[id].finished == false,"zaverchen");
    pubtokens[id].finished = true;
    Polzovateli[0x03C6FcED478cBbC9a4FAB34eF9f40767739D1Ff7].invest_pub += pubtokens[id].value;
  }
  function get_pub (uint id) public { //подтверждение
    require(Polzovateli[msg.sender].role == 4,"ne provaider");
    require(pubtokens[id].finished == false,"zaverchen");
    pubtokens[id].finished = true;
    address investor = pubtokens[id].investor;
    Polzovateli[investor].invest_pub += pubtokens[id].value ;
  }
  mapping (address => bool) Bloked; //блокеровка
  address[] public Bloked_amount;
  function Blok (address investor) public { //функция блокеровки
    require(Polzovateli[msg.sender].role == 2,"ne owner");
    Bloked[investor] = true;
    Bloked_amount.push(investor);
  } 
  function isBloked () public view returns(address[] memory investor){ //функция просмотра блокировки
    return Bloked_amount;
  }
}} 
  function isBloked () public view returns(address[] memory investor){ //функция просмотра блокировки
    return Bloked_amount;
  }
  mapping (uint => obmen) public obmeni; //
  uint public obmen_amount;
  function obmen_token (address adr_to,uint value_pub,uint value_privat) public { //
    require(msg.sender != adr_to,"obmen dlya cebya");
    require(value_privat > 0,"ne vernaya cymma");
    require(value_pub > 0 ,"ne vernaya cymma");
    obmeni[obmen_amount];
  }






  
