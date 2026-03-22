# 🗃️ Redux Toolkit — Appunti Completi di Studio

> Basati sul progetto **Cart App** del corso React di Janis Smilga  
> Documentazione ufficiale: [redux-toolkit.js.org](https://redux-toolkit.js.org/introduction/getting-started)

---

## 📌 Indice

1. [Cos'è Redux Toolkit](#1-cosè-redux-toolkit)
2. [Installazione](#2-installazione)
3. [Architettura generale](#3-architettura-generale)
4. [Setup dello Store](#4-setup-dello-store)
5. [Setup del Provider](#5-setup-del-provider)
6. [Creare uno Slice](#6-creare-uno-slice)
7. [Leggere dati dallo Store — useSelector](#7-leggere-dati-dallo-store--useselector)
8. [Modificare lo stato — useDispatch + Reducers](#8-modificare-lo-stato--usedispatch--reducers)
9. [Reducers: clearCart, removeItem, increase, decrease, calculateTotals](#9-reducers-clearcart-removeitem-increase-decrease-calculatetotals)
10. [Gestire il CarrelloUI — CartContainer e CartItem](#10-gestire-il-carrelloui--cartcontainer-e-cartitem)
11. [Modal con Slice dedicato](#11-modal-con-slice-dedicato)
12. [Async con createAsyncThunk](#12-async-con-createasyncthunk)
13. [extraReducers — sintassi builder](#13-extrareducers--sintassi-builder)
14. [Extras: Redux DevTools + combineReducers](#14-extras-redux-devtools--combinereducers)
15. [Cheatsheet rapido](#15-cheatsheet-rapido)

---

## 1. Cos'è Redux Toolkit

Redux Toolkit (RTK) è la libreria **ufficiale e raccomandata** per gestire lo stato globale in applicazioni React.  
È costruita sopra Redux classico ma elimina il boilerplate ripetitivo.

### Librerie incluse in `@reduxjs/toolkit`

| Libreria        | Ruolo                                                                                         |
| --------------- | --------------------------------------------------------------------------------------------- |
| **redux**       | Core library per la gestione dello stato                                                      |
| **immer**       | Permette di "mutare" lo stato negli slice in modo sicuro (internamente crea copie immutabili) |
| **redux-thunk** | Middleware per gestire azioni asincrone                                                       |
| **reselect**    | Ottimizza e semplifica le funzioni selector                                                   |

> 💡 **`react-redux`** è la libreria separata che _connette_ l'app React allo store Redux (fornisce `Provider`, `useSelector`, `useDispatch`).

---

## 2. Installazione

### Nuovo progetto con template Redux

```sh
npx create-react-app my-app --template redux
# oppure con la versione più recente
npx create-react-app@latest my-app --template redux
```

### Aggiunta a progetto esistente

```sh
npm install @reduxjs/toolkit react-redux
```

---

## 3. Architettura generale

```
┌─────────────────────────────────────────────────┐
│                    REACT APP                     │
│                                                 │
│   Component         Component         Component │
│   useSelector()     useDispatch()               │
│       │                  │                      │
│       ▼                  ▼                      │
│   ┌─────────────────────────────────────────┐   │
│   │               REDUX STORE               │   │
│   │                                         │   │
│   │   cart slice      modal slice   ...     │   │
│   │   { cartItems,    { isOpen }            │   │
│   │     amount,                             │   │
│   │     total,                              │   │
│   │     isLoading }                         │   │
│   └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

**Flusso dati:**

1. Il componente chiama `dispatch(action)` → l'azione arriva al reducer dello slice
2. Il reducer aggiorna lo stato
3. `useSelector()` fa re-render automatico dei componenti che leggono quello stato

---

## 4. Setup dello Store

Lo **store** è il contenitore globale di tutto lo stato dell'app.

```js
// store.js
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./features/cart/cartSlice";

export const store = configureStore({
  reducer: {
    cart: cartReducer, // ogni chiave = "dominio" di stato
  },
});
```

> 🔑 **`configureStore`** sostituisce il vecchio `createStore` di Redux puro.  
> Configura automaticamente Redux DevTools e il middleware `redux-thunk`.

---

## 5. Setup del Provider

Il `Provider` rende lo store disponibile a **tutta** l'albero dei componenti React.  
Va posizionato il più in alto possibile, tipicamente in `index.js`.

```js
// index.js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import App from "./App";
import { store } from "./store";
import { Provider } from "react-redux";

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById("root"),
);
```

---

## 6. Creare uno Slice

Uno **slice** rappresenta una "fetta" dello stato globale relativa a una feature.  
Contiene: nome, stato iniziale, e i reducers.

```js
// features/cart/cartSlice.js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  cartItems: [],
  amount: 0,
  total: 0,
  isLoading: true,
};

const cartSlice = createSlice({
  name: "cart", // prefisso per i nomi delle action (es: 'cart/clearCart')
  initialState,
  reducers: {
    // i reducers vanno qui
  },
});

// console.log(cartSlice) per esplorare: actions, reducer, name...

export default cartSlice.reducer; // esporta solo il reducer per lo store
```

**Struttura cartelle raccomandata:**

```
src/
  features/
    cart/
      cartSlice.js
    modal/
      modalSlice.js
  components/
    Navbar.js
    CartContainer.js
    CartItem.js
    Modal.js
  store.js
```

---

## 7. Leggere dati dallo Store — `useSelector`

`useSelector` è il hook che permette a un componente di **leggere** valori dallo store.  
Si aggiorna automaticamente quando quello stato cambia.

```js
// components/Navbar.js
import { useSelector } from "react-redux";

const Navbar = () => {
  // state = tutto lo store; state.cart = il dominio "cart"
  const { amount } = useSelector((state) => state.cart);

  return (
    <nav>
      <div className="nav-center">
        <h3>redux toolkit</h3>
        <div className="nav-container">
          <CartIcon />
          <div className="amount-container">
            <p className="total-amount">{amount}</p>
          </div>
        </div>
      </div>
    </nav>
  );
};

export default Navbar;
```

> 💡 La funzione passata a `useSelector` si chiama **selector**: riceve `state` e restituisce la parte che ti interessa.

---

## 8. Modificare lo stato — `useDispatch` + Reducers

`useDispatch` restituisce la funzione `dispatch` per inviare **azioni** allo store.

### Come funziona una Action

Internamente una action è un semplice oggetto:

```js
// struttura di una action
const ACTION_TYPE = "cart/clearCart";
const action = { type: ACTION_TYPE, payload: undefined };

// RTK genera queste action creator automaticamente dagli slice
```

### Aggiungere un reducer allo slice

```js
const cartSlice = createSlice({
  name: "cart",
  initialState,
  reducers: {
    clearCart: (state) => {
      state.cartItems = []; // Immer gestisce l'immutabilità automaticamente!
    },
  },
});

// Esportare le action creator generate da RTK
export const { clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

### Usare dispatch in un componente

```js
import { useDispatch } from "react-redux";
import { clearCart } from "../features/cart/cartSlice";

const CartContainer = () => {
  const dispatch = useDispatch();

  return (
    <button
      className="btn clear-btn"
      onClick={() => {
        dispatch(clearCart()); // dispatch chiama l'action creator
      }}
    >
      clear cart
    </button>
  );
};
```

---

## 9. Reducers: clearCart, removeItem, increase, decrease, calculateTotals

Ecco il **cartSlice completo** con tutti i reducers del progetto:

```js
// features/cart/cartSlice.js
import { createSlice } from "@reduxjs/toolkit";
import cartItems from "../../cartItems"; // dati iniziali locali

const initialState = {
  cartItems: [],
  amount: 0,
  total: 0,
  isLoading: true,
};

const cartSlice = createSlice({
  name: "cart",
  initialState,
  reducers: {
    // 1. Svuota completamente il carrello
    clearCart: (state) => {
      state.cartItems = [];
    },

    // 2. Rimuove un item specifico tramite id
    removeItem: (state, action) => {
      const itemId = action.payload;
      state.cartItems = state.cartItems.filter((item) => item.id !== itemId);
    },

    // 3. Aumenta la quantità di un item (payload = { id })
    increase: (state, { payload }) => {
      const cartItem = state.cartItems.find((item) => item.id === payload.id);
      cartItem.amount = cartItem.amount + 1;
    },

    // 4. Diminuisce la quantità di un item (payload = { id })
    decrease: (state, { payload }) => {
      const cartItem = state.cartItems.find((item) => item.id === payload.id);
      cartItem.amount = cartItem.amount - 1;
    },

    // 5. Ricalcola amount totale e totale in $
    calculateTotals: (state) => {
      let amount = 0;
      let total = 0;
      state.cartItems.forEach((item) => {
        amount += item.amount;
        total += item.amount * item.price;
      });
      state.amount = amount;
      state.total = total;
    },
  },
});

export const { clearCart, removeItem, increase, decrease, calculateTotals } =
  cartSlice.actions;

export default cartSlice.reducer;
```

> ⚠️ **Immer e mutazione:** nei reducers di RTK puoi scrivere `state.cartItems = []` come se stessi "mutando" lo stato direttamente. **Immer** intercetta queste operazioni e crea una nuova copia immutabile internamente. Il reducer è ancora puro dal punto di vista di Redux.

### `calculateTotals` va chiamato su ogni cambio al carrello — in `App.js`:

```js
// App.js
import { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import { calculateTotals } from "./features/cart/cartSlice";

function App() {
  const { cartItems } = useSelector((state) => state.cart);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(calculateTotals());
  }, [cartItems]); // si riesegue ogni volta che cartItems cambia

  return (
    <main>
      <Navbar />
      <CartContainer />
    </main>
  );
}
```

---

## 10. Gestire il CarrelloUI — CartContainer e CartItem

### CartContainer.js

```js
import React from "react";
import CartItem from "./CartItem";
import { useDispatch, useSelector } from "react-redux";
import { openModal } from "../features/modal/modalSlice";

const CartContainer = () => {
  const { cartItems, total, amount } = useSelector((state) => state.cart);
  const dispatch = useDispatch();

  // Carrello vuoto
  if (amount < 1) {
    return (
      <section className="cart">
        <header>
          <h2>your bag</h2>
          <h4 className="empty-cart">is currently empty</h4>
        </header>
      </section>
    );
  }

  return (
    <section className="cart">
      <header>
        <h2>your bag</h2>
      </header>
      <div>
        {cartItems.map((item) => {
          return <CartItem key={item.id} {...item} />;
        })}
      </div>
      <footer>
        <hr />
        <div className="cart-total">
          <h4>
            total <span>${total}</span>
          </h4>
        </div>
        {/* Apre il modal di conferma invece di svuotare direttamente */}
        <button
          className="btn clear-btn"
          onClick={() => {
            dispatch(openModal());
          }}
        >
          clear cart
        </button>
      </footer>
    </section>
  );
};

export default CartContainer;
```

### CartItem.js

```js
import React from "react";
import { ChevronDown, ChevronUp } from "../icons";
import { useDispatch } from "react-redux";
import { removeItem, increase, decrease } from "../features/cart/cartSlice";

const CartItem = ({ id, img, title, price, amount }) => {
  const dispatch = useDispatch();

  return (
    <article className="cart-item">
      <img src={img} alt={title} />
      <div>
        <h4>{title}</h4>
        <h4 className="item-price">${price}</h4>
        <button
          className="remove-btn"
          onClick={() => {
            dispatch(removeItem(id)); // payload = id (stringa/numero)
          }}
        >
          remove
        </button>
      </div>
      <div>
        {/* Aumenta quantità */}
        <button
          className="amount-btn"
          onClick={() => {
            dispatch(increase({ id })); // payload = { id }
          }}
        >
          <ChevronUp />
        </button>

        <p className="amount">{amount}</p>

        {/* Diminuisce quantità — se arriva a 1 rimuove l'item */}
        <button
          className="amount-btn"
          onClick={() => {
            if (amount === 1) {
              dispatch(removeItem(id));
              return;
            }
            dispatch(decrease({ id }));
          }}
        >
          <ChevronDown />
        </button>
      </div>
    </article>
  );
};

export default CartItem;
```

> 💡 Nota la differenza nei payload:
>
> - `removeItem(id)` → payload è direttamente `id`
> - `increase({ id })` → payload è un oggetto `{ id }` (perché in futuro potresti aggiungere altri campi)

---

## 11. Modal con Slice dedicato

Il modal di conferma "Vuoi svuotare il carrello?" ha il proprio slice.

### modalSlice.js

```js
// features/modal/modalSlice.js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  isOpen: false,
};

const modalSlice = createSlice({
  name: "modal",
  initialState,
  reducers: {
    openModal: (state) => {
      state.isOpen = true;
    },
    closeModal: (state) => {
      state.isOpen = false;
    },
  },
});

export const { openModal, closeModal } = modalSlice.actions;
export default modalSlice.reducer;
```

### Aggiungere il modalReducer allo store

```js
// store.js
import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./features/cart/cartSlice";
import modalReducer from "./features/modal/modalSlice";

export const store = configureStore({
  reducer: {
    cart: cartReducer,
    modal: modalReducer,
  },
});
```

### Modal.js

```js
import { useDispatch } from "react-redux";
import { closeModal } from "../features/modal/modalSlice";
import { clearCart } from "../features/cart/cartSlice";

const Modal = () => {
  const dispatch = useDispatch();

  return (
    <aside className="modal-container">
      <div className="modal">
        <h4>Remove all items from your shopping cart?</h4>
        <div className="btn-container">
          {/* Conferma: svuota carrello E chiude modal */}
          <button
            type="button"
            className="btn confirm-btn"
            onClick={() => {
              dispatch(clearCart());
              dispatch(closeModal());
            }}
          >
            confirm
          </button>
          {/* Annulla: chiude solo il modal */}
          <button
            type="button"
            className="btn clear-btn"
            onClick={() => {
              dispatch(closeModal());
            }}
          >
            cancel
          </button>
        </div>
      </div>
    </aside>
  );
};

export default Modal;
```

### Rendering condizionale del Modal in App.js

```js
// App.js
function App() {
  const { cartItems, isLoading } = useSelector((state) => state.cart);
  const { isOpen } = useSelector((state) => state.modal);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(calculateTotals());
  }, [cartItems]);

  if (isLoading) {
    return (
      <div className="loading">
        <h1>Loading...</h1>
      </div>
    );
  }

  return (
    <main>
      {isOpen && <Modal />} {/* Il modal appare solo se isOpen = true */}
      <Navbar />
      <CartContainer />
    </main>
  );
}
```

> 🏗️ **Principio:** ogni feature ha il suo slice. Lo store aggrega tutti i reducer. L'app diventa modulare e scalabile.

---

## 12. Async con `createAsyncThunk`

Quando devi **fetchare dati da un'API**, usi `createAsyncThunk`.  
Gestisce automaticamente i 3 stati del ciclo di vita asincrono: `pending`, `fulfilled`, `rejected`.

### Ciclo di vita di una async action

```
dispatch(getCartItems())
       │
       ▼
  [pending]   → isLoading = true
       │
       ▼
  fetch API...
       │
   ┌───┴────┐
   ▼        ▼
[fulfilled] [rejected]
isLoading   isLoading
 = false     = false
cartItems   (gestisci
 = data      l'errore)
```

### createAsyncThunk — versione base con fetch

```js
// features/cart/cartSlice.js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

const url = "https://www.course-api.com/react-useReducer-cart-project";

export const getCartItems = createAsyncThunk(
  "cart/getCartItems", // nome dell'action type (prefisso/nome)
  () => {
    return fetch(url)
      .then((resp) => resp.json())
      .catch((err) => console.log(err));
  },
);
```

### createAsyncThunk — versione avanzata con axios e `thunkAPI`

```js
import axios from "axios";

export const getCartItems = createAsyncThunk(
  "cart/getCartItems",
  async (name, thunkAPI) => {
    // name = eventuale argomento passato a dispatch(getCartItems(arg))
    // thunkAPI = oggetto con metodi utili:
    //   thunkAPI.getState()      → leggi lo stato attuale
    //   thunkAPI.dispatch(...)   → dispatch altre azioni
    //   thunkAPI.rejectWithValue → invia un errore personalizzato

    try {
      const resp = await axios(url);
      return resp.data; // questo diventa action.payload in 'fulfilled'
    } catch (error) {
      return thunkAPI.rejectWithValue("something went wrong");
    }
  },
);
```

> 💡 `thunkAPI.rejectWithValue('messaggio')` → il payload dell'azione `rejected` sarà `'messaggio'` invece dell'errore grezzo. Utile per gestire messaggi di errore nel reducer.

### Chiamare il thunk nell'app

```js
// App.js
useEffect(() => {
  dispatch(getCartItems()); // si chiama una volta al mount
}, []);
```

---

## 13. `extraReducers` — sintassi builder

Gli `extraReducers` gestiscono le azioni asincrone generate da `createAsyncThunk`.  
La **sintassi builder** è quella raccomandata (la sintassi con oggetto `{}` è deprecata).

```js
const cartSlice = createSlice({
  name: "cart",
  initialState,
  reducers: {
    // ...reducers sincroni...
  },

  // extraReducers per azioni esterne (createAsyncThunk, ecc.)
  extraReducers: (builder) => {
    builder
      .addCase(getCartItems.pending, (state) => {
        state.isLoading = true;
      })
      .addCase(getCartItems.fulfilled, (state, action) => {
        state.isLoading = false;
        state.cartItems = action.payload; // i dati fetchati dall'API
      })
      .addCase(getCartItems.rejected, (state, action) => {
        console.log(action);
        state.isLoading = false;
        // action.payload = valore passato a rejectWithValue()
      });
  },
});
```

### Differenza tra `reducers` e `extraReducers`

|                    | `reducers`                         | `extraReducers`                                  |
| ------------------ | ---------------------------------- | ------------------------------------------------ |
| **Usa per**        | Azioni sincrone interne allo slice | Azioni asincrone (thunk) o azioni di altri slice |
| **Action creator** | Generato automaticamente da RTK    | Definito separatamente con `createAsyncThunk`    |
| **Syntax**         | `clearCart: (state) => {...}`      | `builder.addCase(action, handler)`               |

---

## 14. Extras: Redux DevTools + combineReducers

### Redux DevTools

Installa l'estensione browser **Redux DevTools** (Chrome/Firefox).  
`configureStore` di RTK la abilita automaticamente in sviluppo — zero configurazione!

Con DevTools puoi:

- Visualizzare ogni action dispatched con il suo payload
- Vedere come cambia lo stato dopo ogni action
- "Time-travel": tornare indietro a stati precedenti
- Exportare / importare lo stato

### combineReducers

Con più slice, `configureStore` usa internamente `combineReducers` automaticamente:

```js
// Questo è equivalente a:
import { combineReducers } from "@reduxjs/toolkit";

const rootReducer = combineReducers({
  cart: cartReducer,
  modal: modalReducer,
});

const store = configureStore({ reducer: rootReducer });
```

> RTK gestisce tutto questo automaticamente quando passi un oggetto a `reducer: {}`. Non devi chiamare `combineReducers` manualmente nella maggior parte dei casi.

---

## 15. Cheatsheet rapido

### Setup minimo

```
1. npm install @reduxjs/toolkit react-redux
2. Crea store.js → configureStore({ reducer: {} })
3. Wrappa App con <Provider store={store}>
4. Crea features/nomeFeature/nomeSlice.js → createSlice(...)
5. Aggiungi il reducer allo store
6. useSelector per leggere, useDispatch per scrivere
```

### Pattern Dispatch

```js
// Azione sincrona
dispatch(clearCart());
dispatch(removeItem(id));
dispatch(increase({ id }));

// Azione asincrona
dispatch(getCartItems());
```

### Pattern Selector

```js
const { cartItems, amount, total } = useSelector((state) => state.cart);
const { isOpen } = useSelector((state) => state.modal);
```

### Anatomy di uno Slice

```js
createSlice({
  name: 'nomeSlice',           // prefisso action types
  initialState,                // stato iniziale
  reducers: { ... },           // azioni sincrone
  extraReducers: (builder) => { // azioni asincrone (thunk)
    builder.addCase(...)
  }
})
```

### Anatomy di un Thunk

```js
export const myThunk = createAsyncThunk(
  "slice/actionName", // action type
  async (arg, thunkAPI) => {
    // callback asincrona
    const data = await fetch(url);
    return data; // → action.payload in fulfilled
  },
);
```

---

## 🔗 Risorse

- [Redux Toolkit Docs](https://redux-toolkit.js.org/introduction/getting-started)
- [React Course Udemy — Janis Smilga](https://www.udemy.com/course/react-tutorial-and-projects-course/?referralCode=FEE6A921AF07E2563CEF)
- [Course API](https://www.course-api.com/)
- [Hero Icons](https://heroicons.com/)
