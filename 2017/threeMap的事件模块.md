 ```javascript
 function classEvent() {  //构造函数  这里相当于一个类   this既是classEvent对象本身
            this.eventList = {
                length: 0
            }
            this.addEvent = function (name, value) {
                Object.defineProperty(this.eventList, name, {
                    value: value,
                    enumerable: true,
                    configurable: true
                });
                this.eventList.length++;
            }
            this.addOneEvent = function (name, value) {
                Object.defineProperty(this.eventList, name, {
                    value: function (ans, event) {
                        value(ans, event);
                        delete this[name];
                        this.length--;
                    },
                    enumerable: true,
                    configurable: true
                });
                this.eventList.length++;
            }
            this.removeEvent = function (name) {
                if (this.eventList.hasOwnProperty(name)) {
                    delete this.eventList[name];
                    this.eventList.length--;
                }
            }
            this.run = function (ans, event) {
                for (var key in this.eventList) {
                    if (key == 'length')
                        continue;
                    this.eventList[key](ans, event);
                }
            }
        }

        this.onchangePreOffice = new classEvent(); // new 运算符创建一个用户定义的对象类型的实例     此处的this为threeMap   

}

```
