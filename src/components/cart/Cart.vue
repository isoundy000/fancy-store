<template>

	<div class="page">
		<headers :tabname="$t('m.HeaderCart')"></headers>
		<transition :name="slidename">
			<div class="container" v-show="mainarea">
				<div v-show="!havePage">
					<nopage></nopage>
				</div>
				<div v-show="havePage">
					<div class="cart-item" :class="{selected: itemIndex === cartIndex}" v-for="(cartItem,cartIndex) in $store.state.carts" :key="cartIndex" @click="onDetail(cartItem)">
						<v-touch @swipeleft="onSwipeLeft(cartIndex)" @swiperight="onSwipeRight(cartIndex)">
							<div class="cart-content flex-align-center">
								<div class="goods-radio" @click.stop="onGoodsRadio(cartItem)">
                  <i class="cart_radio" v-show="!cartItem.goodsRadio" ></i>
                  <i class="cart_radio_select" v-show="cartItem.goodsRadio"></i>
								</div>
								<div class="flex">
									<div class="goods-img">
										<img v-lazy="cartItem.GoodsImage" />
									</div>
									<div class="goods-textBox">
										<p class="goods-name">{{cartItem.GoodsName}}</p>
										<div class="goodsOp flex">
                      <i class="shop_cut" @click.stop="onCutCart(cartItem)"></i>
											<input type="text" :value="cartItem.GoodsNum" />
                      <i class="shop_add" @click.stop="onAddCart(cartItem)"></i>
										</div>
										<p class="goods-coach">¥{{cartItem.GoodsPrice}}</p>
									</div>
								</div>
							</div>
							<!--v-show="itemIndex === cartIndex"-->
							<div class="remove" @click.stop="onRemove(cartItem)">
               <i></i>
               </div>
						</v-touch>
					</div>
				</div>

			</div>
		</transition>

		<div class="cartBottom-detail flex-between" v-show="$store.state.carts" v-cloak>
			<div class="flex">
				<div class="shopRadio" @click="onSelectAll()">
          <i class="cart_radio" v-show="!goodsRadioAll" ></i>
          <i class="cart_radio_select" v-show="goodsRadioAll"></i>
				</div>
				<div class="bottom-left">
					<p>合计: <span class="shopCoach">¥{{allCoach}}</span> </p>
				</div>
			</div>

			<div class="subminCart" @click="onOrder">
				<p>提交订单</p>
			</div>
		</div>
		<footers :urlRouter="$route.path"></footers>
	</div>

</template>

<script>
import Headers from "../base/Header.vue";
import Footers from "../base/Footer.vue";
import Nopage from "../base/NoPage.vue";
import { mapGetters, mapMutations } from "vuex";
export default {
  data() {
    return {
      goodsRadioAll: false,
      allCoach: 0,
      radioArr: [],
      havePage: false,
      itemIndex: "",
      slidename: "slide-go",
      mainarea: false
    };
  },
  components: {
    Headers,
    Footers,
    Nopage
  },
  computed: {
    ...mapGetters(["this.$store.state.carts", "this.$store.state.comname"])
  },
  mounted() {
    const that = this;
    that.mainarea = true;
    if (that.$store.state.carts.length == 0) {
      that.havePage = false;
    } else {
      that.havePage = true;
      that.$store.state.carts.forEach(function(item) {
        if (typeof item.goodsRadio == "undefined") {
          that.$set(item, "goodsRadio", false);
        }
      });
    }
    /*判断动画是进还是出*/
    if (
      this.$store.state.comname === "index" ||
      this.$store.state.comname === "category" ||
      this.$store.state.comname === "goodsdetail"
    ) {
      this.slidename = "slide-go";
    } else {
      this.slidename = "slide-back";
    }
    this.setComname("cart");
  },
  methods: {
    /*选择单个商品*/
    onGoodsRadio(item) {
      const that = this;
      that.radioArr = [];
      item.goodsRadio = !item.goodsRadio;
      that.$store.state.carts.forEach(function(itemGoods) {
        that.radioArr.push(itemGoods.goodsRadio);
      });
      if (that.radioArr.indexOf(false) == -1) {
        that.goodsRadioAll = true;
      } else {
        that.goodsRadioAll = false;
      }
      that.onCalAllCoach();
    },
    /*选择全部商品*/
    onSelectAll() {
      this.goodsRadioAll = !this.goodsRadioAll;
      if (this.goodsRadioAll) {
        this.$store.state.carts.forEach(function(item) {
          item.goodsRadio = true;
        });
      } else {
        this.$store.state.carts.forEach(function(item) {
          item.goodsRadio = false;
        });
      }
      this.onCalAllCoach();
    },
    /*添加商品数量*/
    onAddCart(item) {
      item.GoodsNum++;
      this.onCalAllCoach();
    },
    /*减少商品数量*/
    onCutCart(item) {
      if (item.GoodsNum > 1) {
        item.GoodsNum--;
        this.onCalAllCoach();
      }
    },
    /*计算价格*/
    onCalAllCoach() {
      const that = this;
      that.allCoach = 0;
      that.$store.state.carts.forEach(function(item) {
        if (item.goodsRadio) {
          that.allCoach += item.GoodsNum * item.GoodsPrice;
        }
      });
    },
    /*删除商品*/
    onRemove(item) {
      let index = this.$store.state.carts.indexOf(item);
      this.$store.state.carts.splice(index, 1);
      this.itemIndex = "";
    },
    /*提交订单*/
    onOrder() {
      let orderArr = [];
      this.$store.state.carts.forEach(function(item) {
        if (item.goodsRadio) {
          orderArr.push(item);
        }
      });
      if (orderArr.length) {
        this.$router.push("orderwait");
        this.setOrders(orderArr);
      }
    },
    /*向左滑出现删除*/
    onSwipeLeft(index) {
      this.itemIndex = index;
    },
    /*向右滑隐藏删除*/
    onSwipeRight() {
      this.itemIndex = "";
    },
    /*进入商品详情*/
    onDetail(item) {
      this.$router.push("/detail");
      this.setGoods(item);
    },
    ...mapMutations({
      setOrders: "SET_ORDERS",
      setGoods: "SET_GOODS",
      setComname: "SET_COMNAME"
    })
  }
};
</script>

<style lang="less" scoped>
@import "../../../static/less/variable.less";
.container {
  padding-bottom: 1.6rem;
  overflow-x: hidden;
}

.cart-item {
  border-bottom: 1px solid #cccccc;
  padding-top: 0.2rem;
  height: 2.4rem;
  -webkit-transition: all 0.3s linear;
  -webkit-user-select: none;
  position: relative;
}

.remove {
  background: linear-gradient(90deg, #9bbeff 0%, #6495ed 100%);
  width: 18%;
  height: 2.6rem;
  line-height: 2.6rem;
  text-align: center;
  text-decoration: none;
  position: absolute;
  right: -18%;
  top: 0;
  i {
    position: absolute;
    top: 0;
    bottom: 0;
    margin: auto;
    left: 20%;
    width: 0.4rem;
    height: 0.48rem;
    background: url('../../../static/img/icon/common_sprites.png') -10px -438px;
  }
}

.cart-content {
  width: 100%;
}

.selected {
  -webkit-transform: translate(-12%, 0);
  -webkit-transition: all 0.3s linear;
}

.goods-radio {
  margin: 0 0.2rem;
  width: 0.34rem;
  height: 0.34rem;
  i {
    display: inline-block;
    width: 100%;
    height: 100%;
  }
  
}
.cart_radio{
    background: url('../../../static/img/icon/common_sprites.png') -10px -114px;

  }
  .cart_radio_select{
    background: url('../../../static/img/icon/common_sprites.png') -10px -10px;
    
  }
.goods-img {
  margin-right: 0.2rem;
  width: 2rem;
  height: 2rem;
  img {
    width: 100%;
    height: 100%;
  }
}

.goods-name {
  font-size: 0.28rem;
}

.goods-coach {
  font-size: 0.3rem;
  color: red;
  align-self: flex-end;
}

.goodsOp {
  margin: 0.4rem 0;
  i {
    width: 0.48rem;
    height: 0.48rem;
  }
  input {
    border: none;
    font-size: 0.36rem;
    text-align: center;
    width: 0.6rem;
  }
   .shop_cut{
    
    background: url('../../../static/img/icon/common_sprites.png') -10px -574px;
  }
  .shop_add{
    background: url('../../../static/img/icon/common_sprites.png') -10px -642px;
  }
}

.cartBottom-detail {
  height: 0.8rem;
  line-height: 0.8rem;
  font-size: 0.28rem;
  width: 100%;
  position: absolute;
  bottom: 0.8rem;
  color: @base_color;
  background: @theme_background;
  /*background:red;*/
  i {
    display: inline-block;
    vertical-align: middle;
    margin: 0 0.2rem;
    width: 0.3rem;
    height: 0.36rem;
  }
}

.subminCart {
  padding-right: 0.2rem;
}
</style>