<template>
    <input
        type="text"
        ref="inp"
        v-model="input"
        @keydown="CodEnter"
        class="none"
    />

    <div class="m-3">
        <button class="btn btn-primary" @click="SkanFocus">Сканировать</button>
    </div>

    <h3 class="m-3">Название: {{ product.title }}</h3>
    <h3 class="m-3">Цена: {{ product.price }}</h3>

    <div class="w-25 m-3">
        <div class="form-floating mb-3">
            <input
                type="text"
                ref="title"
                @keydown="FocusInput($event, 'code')"
                class="form-control"
                :class="{ 'is-invalid': errors.title }"
                id="#"
                placeholder="#"
                v-model="title"
            />
            <label for="#">Название</label>
            <div v-if="errors.title" class="invalid-feedback">
                {{ errors.title.join('. ') }}
            </div>
        </div>
        <div class="form-floating mb-3">
            <input
                @keydown="FocusInput($event, 'price')"
                ref="code"
                :class="{ 'is-invalid': errors.code }"
                v-model="code"
                type="text"
                class="form-control"
                id="##"
                placeholder="#"
            />
            <label for="##">Код товар</label>
            <div v-if="errors.code" class="invalid-feedback">
                {{ errors.code.join('. ') }}
            </div>
        </div>
        <div class="form-floating mb-3">
            <input
                v-model="price"
                :class="{ 'is-invalid': errors.price }"
                ref="price"
                type="number"
                class="form-control"
                id="###"
                placeholder="#"
                @keydown="BtnCreate"
            />
            <label for="###">Цена</label>
            <div v-if="errors.price" class="invalid-feedback">
                {{ errors.price.join('. ') }}
            </div>
        </div>

        <button class="btn btn-primary w-100 btn-lg" @click="CreateProduct">
            Отправить
        </button>
    </div>

    <div class="p-3">
        <table class="table-bordered table">
            <thead>
                <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Название</th>
                    <th scope="col">Цена</th>
                    <th scope="col">Код товара</th>
                    <th scope="col">Дийствие</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="product in products">
                    <th>{{ product.id }}</th>
                    <td>{{ product.title }}</td>
                    <td>{{ product.price }} руб.</td>
                    <td>{{ product.code }}</td>
                    <td>
                        <button
                            class="btn btn-danger"
                            @click="DelProduct(product.id)"
                        >
                            <i class="bi bi-trash"></i>
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <div v-if="is_long" class="long">
        <i class="bi bi-arrow-repeat ico"></i>
    </div>
</template>

<script>
export default {
    name: 'App',

    data() {
        return {
            input: '',
            APISever: 'http://127.0.0.1:8000/api',
            title: '',
            code: '',
            price: '',
            errors: [],
            products: [],
            is_long: false,
            product: {
                title: '',
            },
        };
    },

    mounted() {
        document.body.setAttribute('data-bs-theme', 'dark');
        this.GetProducts();
    },

    methods: {
        SkanFocus() {
            this.$refs.inp.focus();
        },

        FocusInput(event, from) {
            if (event.key == 'Enter') {
                this.$refs[from].focus();
            }
        },

        DelProduct(id) {
            this.server('/product/' + id, 'DELETE')
                .then((result) => {
                    this.products = result.products;
                })
                .catch((error) => console.log('error', error));
        },

        CodEnter(event) {
            if (event.key == 'Enter') {
                this.is_long = true;
                let formdata = new FormData();
                formdata.append('code', this.input);
                this.server('/productcode', 'POST', formdata)
                    .then((result) => {
                        if (result.product) {
                            this.product = result.product;
                        } else {
                            this.product = { title: 'товар не найден' };
                        }
                        setTimeout(() => {
                            this.input = '';
                            this.is_long = false;
                        }, 500);
                    })
                    .catch((error) => console.log('error', error));
            }
        },

        BtnCreate(event = null) {
            if (event) {
                if (event.key == 'Enter') {
                    this.CreateProduct();
                      this.$refs.title.focus();
                }
            } else {
                this.CreateProduct();
            }
        },
//  ываыва
        GetProducts() {
            this.server('/product', 'GET')
                .then((result) => {
                    this.products = result.products;
                })
                .catch((error) => console.log('error', error));
        },

        CreateProduct() {
            let formdata = new FormData();
            formdata.append('title', this.title);
            formdata.append('code', this.code);
            formdata.append('price', this.price);
            this.server('/product', 'POST', formdata)
                .then((result) => {
                    if (result.errors) {
                        this.errors = result.errors;
                    } else {
                        this.products = result.products;
                        this.errors = [];
                        this.title = '';
                        this.code = '';
                        this.price = '';
                    }
                })
                .catch((error) => console.log('error', error));
        },

        async server(route, method = 'GET', formdata = null) {
            let myHeaders = new Headers();
            myHeaders.append('Accept', 'application/json');

            let requestOptions = {
                method: method,
                headers: myHeaders,
                redirect: 'follow',
            };

            if (method != 'GET' && method != 'DELETE') {
                requestOptions.body = formdata;
            }

            return await fetch(this.APISever + route, requestOptions).then(
                (response) => response.json(),
            );
        },
    },
};
</script>
