<script setup>
</script>

<template>
  <div class="border-box cart-control" @click="cart_open()">
    <i class="bi bi-handbag"></i><br>
    <span>Giỏ hàng</span>
  </div>
  <Teleport to="#sidebar">
    <Transition>
      <div class="overlay" v-if="cart_show" @click="cart_show = false"></div>
    </Transition>
    <Transition>
      <div class="cart" id="cart" v-if="cart_show">
        <div class="cart-header">
          <p>
            <i class="bi bi-bag"></i>
            Giỏ hàng
          </p>
          <div class="cart-close-icon">
            <i @click="cart_close" class="bi bi-box-arrow-right"></i>
          </div>
        </div>
        <div class="cart-body">
          <div v-if="store.cart_list == null || store.cart_list.length == 0" class="cart-empty">
            <div class="cart-title">
              <h3>Giỏ hàng của bạn đang trống!</h3>
              <p>Hãy thêm sản phẩm vào giỏ hàng của bạn.</p>
            </div>
            <div class="cart-image">
              <img src="/images/cart-icon.webp" alt="">
            </div>
          </div>
          <div v-if="store.cart_list != null && store.cart_list.length > 0" class="cart-list">
            <div v-for="(cart, index) in store.cart_list" :key="index" class="cart-item row">
              <div class="col-3 cart-item-image">
                <img :src="cart.item.image" :alt="cart.item.name">
              </div>
              <div class="col-6 cart-item-content">
                <h6>{{ cart.item.name }}</h6>
                <div class="cart-product-control">
                  <button id="descProduct-btn" @click="descProduct(cart.item.id)">-</button>
                  <input type="text" :value="cart.quantity" disabled>
                  <button id="inscProduct-btn" @click="inscProduct(cart.item.id)">+</button>
                </div>
                <p class="remove-cart-item" @click="removeCart(cart.item.id)">Xóa khỏi giỏ hàng</p>
              </div>
              <div class="col-3">
                <h6>{{ (cart.item.price * cart.quantity).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".") }}đ</h6>
              </div>
            </div>
          </div>
        </div>
        <div class="cart-footer">
          <div v-if="store.cart_list != null && store.cart_list.length > 0" class="checkout-form">
            <div class="form-group">
              <label for="checkout_name">Tên liên hệ: </label>
              <input type="text" placeholder="Họ và tên" id="checkout_name">
            </div>
            <div class="form-group">
              <label for="checkout_name">Số điện thoại: </label>
              <input type="text" placeholder="Nhập số điện thoại" id="checkout_name">
            </div>
            <h4>Tổng đơn: {{ totalPay }}</h4>
            <button id="checkout-cart-btn" class="btn">Lên đơn giỏ hàng</button>
          </div>
          <div class="cart-close-lable">
            <p @click="cart_close">Đóng giỏ hàng</p>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>
<script>
import { useCartStore } from '../stores/cart'
export default {
  data() {
    return {
      store: useCartStore(),
      cart_show: false,
    }
  },
  methods: {
    cart_open() {
      this.cart_show = true
    },
    cart_close() {
      this.cart_show = false
    },
    removeCart(id) {
      this.store.remove(id)
    },
    inscProduct(id) {
      this.store.increaseItemQuantity(id);
    },
    descProduct(id) {
      this.store.decreaseItemQuantity(id);
    },
    loadCookie() {
      const cookieValue = this.$cookies.get('cart')
      if (cookieValue != null) {
        this.store.cart_list = cookieValue;
      }
    },
  },
  computed: {
    totalPay() {
      let total = 0;
      for (let i = 0; i < this.store.cart_list.length; i++) {
        const p = parseInt(this.store.cart_list[i].item.price.toString().replace(/\D/g, ''));
        total += p * this.store.cart_list[i].quantity;
      }
      return total.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".") + " ₫";
    }
  },
  mounted() {
    this.loadCookie()
  },
}
</script>
<style lang="scss" scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 2;
}

.cart-control {
  cursor: pointer;
}

.cart.open {
  opacity: 1;
  transform: translateX(0);
}

.cart {
  width: 410px;
  height: 100vh;
  background-color: white;
  position: fixed;
  right: 410px;
  top: 0;
  z-index: 3;
  transform: translateX(100%);
  transition: opacity 0.3s ease, transform 0.3s ease;

  &-header {
    border-bottom: 1px solid rgba(0, 0, 0, 0.24);
    padding: 20px 15px;

    p {
      font-size: 15px;
      margin-bottom: 0 !important;

      i {
        font-size: 19px;
        margin-right: 5px;
      }
    }
  }

  &-body {
    padding: 20px 15px;
    overflow-y: auto;
    height: calc(100vh - 300px);

    .cart-title {
      padding: 50px 20px;

      h3 {
        font-size: 25px;
      }

      p {
        font-size: 14px;
        color: rgba(0, 0, 0, 0.678);
      }
    }

    .cart-product-control {
      display: flex;

      input {
        width: 30px;
        text-align: center;
        font-size: 13px;
        margin: 0 5px;
      }

      button {
        border: none;
      }
    }

    .cart-item {
      margin-bottom: 15px;

      &:not(:last-child) {
        border-bottom: 1px solid rgba(0, 0, 0, 0.24);
        padding-bottom: 15px;
      }

      .cart-item-content {
        display: flex;
        flex-direction: column;
        justify-content: space-between;

        p {
          margin-bottom: 0 !important;
        }
      }

      img {
        border-radius: 5px;
        width: 100%;
        height: 96px;
        object-fit: cover;
      }

      .remove-cart-item {
        font-size: 14px;
        cursor: pointer;
      }

    }

    img {
      width: 100%;
      height: auto;
      object-fit: cover;
    }
  }

  &-footer {
    margin-top: 20px;

    .checkout-form {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      width: 100%;

      .form-group {
        margin-bottom: 10px;
        display: flex;
        justify-content: space-between;
        width: 66%;
        font-size: 14px;

        label {
          margin-right: 10px;
        }

        input {
          padding: 0 0 0 10px;
        }
      }

      #checkout-cart-btn {
        margin: 0 auto;
        width: 200px;
        padding: 10px 0;
        font-size: 15px;
        color: white;
        background: #746b61;
      }
    }
  }

  &-close-icon {
    position: absolute;
    top: 20px;
    right: 20px;

    i {
      font-size: 20px;
      cursor: pointer;
    }
  }

  &-close-lable {
    position: absolute;
    bottom: 5px;
    left: 50%;

    p {
      font-size: 13px;
      padding-bottom: 3px;
      cursor: pointer;
      border-bottom: 1px solid rgba(0, 0, 0, 0.356);
      transform: translate(-50%, 0);
    }
  }
}
</style>
