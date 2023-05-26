### 变量的初始化也有讲究
1. 变量名简单易懂，简洁明了
2. 使用const let替代var
3. 常量请使用const，对象数组如果引用不会改变也请使用const
4. 初始化对象时，请提前声明好属性值以及默认值（v8引擎隐藏类的优化）
5. 非极端情况下请不要使用delete语法删除属性值，可声明新变量或者设置属性值为undefined进行替代(v8引擎隐藏类优化)

       // bad
       const a = 1
    
       // good
       const num = 1

       //bad
       var num = 1
       let notChangeVar = 2
       let arr = [1,2,3]
       arr.push(4)
    
       // good
       const num = 1
       const notChangeVar = 2
       const arr = [1,2,3]
       arr.push(4)

       //bad
       let payload = {}
       payload.location = {}
       payload.type = this.data.options.type
       payload.title = `${parent ? `${parent.title}-` : ''}${item.title}`
       payload.location.latitude = item.location.lat
       payload.location.longitude = item.location.lng
       payload.id = item.id
       payload.custom = true      

       // good
       const payload = {
          location: {},
          type: this.data.options.type,
          title: `${parent ? `${parent.title}-` : ''}${item.title}`,
          //...
       }

### 善用策略模式
    // bad
    <el-form-item label="金额" prop="name" v-if="type==1 ||type==3 || type==5"/>
    
    <template slot-scope="scope">
      <span v-if="scope.row.tradeType === 0">线下充值</span>
      <span v-if="scope.row.tradeType === 1">支付宝</span>
      <span v-if="scope.row.tradeType === 2">微信</span>
    </template>
    
    // good
    <el-form-item label="金额" prop="name" v-if="[1,3,5].includes(type)"/>
    
    <template slot-scope="scope">
      <span>{{state.tradeTypeArray[scope.row.tradeType]}}</span>
    </template>
    
    const state = reactive({
      tradeTypeArray: ['线下充值', '支付宝' , '微信']
    })

### 善用三目运算符
    // bad
    if (state.detailData.status == 0) {
      state.viewstyle = 'check'
    } else {
      state.viewstyle = 'view'
    }
    
    // good
    state.viewstyle = state.detailData.status === 0 ? 'check' : 'view'

### async await 替换.then.catch
      // bad
      this.$confirm('二次确认！', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const [err, res] = await awaitTo(changegetAudit(params));
        if (err) return;
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        });
      });
      
      // good
      try {
        await this.$confirm('二次确认！', {
          confirmButtonText: '确认',
          cancelButtonText: '取消',
          type: 'warning'
        })
        const [err, res] = await awaitTo(changegetAudit(params));
        if (err) return;
      } catch (err) {
        this.$message({
          type: 'info',
          message: '已取消'
        });
      }

### 禁止使用缩写变量名
    // bad
    async handleClick(e) {..}
    async comfirm(c){...}
    
    // good
    async handleClick(event) {..}
    async comfirm(config){...}

### 善用解构赋值
    // bad
    const startTime = timeArray[0]
    const endTime = timeArray[1]
    
    // good
    const [startTime, endTime] = timeArray
    
    // bad
    if (this.detailData['signTime'] && this.detailData['serviceExpireTime']) {
      if (new Date(this.detailData['signTime']).getTime() >= new Date(this.detailData['serviceExpireTime']).getTime()) {
        this.$message.error('合同签约时间必须小于合同到期时间');
        return;
      }
    }
    
    // good
    const { signTime, serviceExpireTime } = this.detailData
    
    if (signTime && serviceExpireTime) {
      if (new Date(signTime).getTime() >= new Date(serviceExpireTime).getTime()) {
        this.$message.error('合同签约时间必须小于合同到期时间');
        return;
      }
    }

### 善用every,map,reduce,some,filter替换forEach或者for循环
面向函数式编程而不是面向命令式编程,过来人的推荐。

1. 数组元素改变请使用map
2. 数组增加元素请使用concat
3. 数组元素删除请使用filter
4. 数组元素满足某条件取布尔值请使用some,every
5. 什么上面都搞不定，请使用万能的reduce

const numbers = [1, 2, 3, 4, 5];
     
    // bad
    let sum = 0;
    for (let num of numbers) {
      sum += num;
    }
    sum === 15;
    
    // good
    let sum = 0;
    numbers.forEach(num => sum += num);
    sum === 15;
    
    // best (use the functional force)
    const sum = numbers.reduce((total, num) => total + num, 0);
    sum === 15;

    // bad
    const increasedByOne = [];
    for (let i = 0; i < numbers.length; i++) {
      increasedByOne.push(numbers[i] + 1);
    }
    
    // good
    const increasedByOne = [];
    numbers.forEach(num => increasedByOne.push(num + 1));
    
    // best (keeping it functional)
    const increasedByOne = numbers.map(num => num + 1);

### 函数应该尽可能为纯函数
何谓纯函数？固定的输入，固定的输出。

**在函数中，尽量或者杜绝直接去修改传入的参数**

    // bad
    const addItemToCart = (cart, item) => {
      cart.push({item, date: Date.now()})
    }
    let test = []
    addItemToCart(test, 'first')
    
    // good
    const addItemToCart = (cart, item) => {
      return [...cart, {item, date: Date.now()}]
    }
    let test = []
    test = addItemToCart(test, 'first')

### prop推荐加上默认值以及类型
    // bad
    defineProps({
      id: String,
      data: Array,
      obj: Object
    })
    
    // good
    defineProps({
      id: {
        type: String;
        default: ''
      },
      data: {
        type: Array,
        default: () => ([])
      },
      obj: {
        type: Object,
        default: () => ({})
      }
    })

### 减少if嵌套
1. 赋值使用三目运算符
2. 尽量使用短路运算符
3. 能今早return出去的逻辑，提前返回出去

       // bad
       let a;
       if(x === 1) {
         a = 1
       }else {
         a = 2
       }
       // good
       const a = x === 1 ? 1 : 2

    // bad
    if(a===1) {
    if(b ===2) {
    //...
    }
    }
    
    // good
    if(a===1 && b ===2) {
    //...
    }
