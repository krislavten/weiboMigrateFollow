# weiboMigrateFollow
微博迁移关注列表 / 目前需要前端出才能用

## 背景故事
前天因为在网上骂湖北红十字会，用了八年的微博被封了 还是永久封，很无语
封号不要紧，本来就没少粉丝 但是我1000个关注也就丢了，所以在网上搜了一下脚本，果然 我发下了这个项目 [weiboBathFollow](https://github.com/nondanee/weiboBatchFollow) 但是已经不能用了，我改了一下，目前还是不太可用的阶段，后面我会优化一下

## 使用
### 拉取账户关注列表

1. 前往微博 H5 版首页 ([https://m.weibo.cn](https://m.weibo.cn))
2. 登录 A 账号
3. 右键 "检查" 打开开发者工具，复制下面的代码，粘贴到控制台，等待结果并复制

```javascript
// get follow list by containerid
// need replace containerid value
(() => {
	let uid = config.uid, follows = []
	const pageQuery = (page = 1) => {
		let xhr = new XMLHttpRequest()	
		xhr.onreadystatechange = () => {
			if(xhr.readyState == 4 && xhr.status == 200){
        let data = JSON.parse(xhr.responseText).data
          // page numbers = follows/20 
          // my account have 1007 follows
          if(page >= 51) return console.log(JSON.stringify(follows))
          if (data.cards[data.cards.length - 1] &&  Array.isArray(data.cards[data.cards.length - 1].card_group)) {
            data.cards[data.cards.length - 1].card_group.forEach(card => follows.push(card.user.id))
            setTimeout(pageQuery(page + 1), 500)
          }
			}
    }
    // containerid 可以看下个人主页点击进入关注列表上面的 query 参数
		xhr.open('GET', `https://m.weibo.cn/api/container/getIndex?containerid=xxxx-_selffollowed&page=${page}`)
		xhr.send()
	}
	pageQuery()
})()
```
4. 复制一下控制台上打印出来的数组，最好是保存在一个 js 文件中

### 按列表为 B 账号添加关注
1. 退出 A 账号，登录 B 账号
2. 右键 "检查" 打开开发者工具，复制下面的代码并填入上一步得到的列表(不要超过150个)，粘贴到控制台，等待全部关注完成
```javascript
// follow user by userid 
// maybe only 100 users can be follow in one day
// need to replace the 'st' with XSRF-TOKEN in cookies

(list => {
	let index = -1, length = list.length
	const follow = uid => {
		let xhr = new XMLHttpRequest()
		xhr.onreadystatechange = () => {
			if(xhr.readyState == 4 && xhr.status == 200){
				let data = JSON.parse(xhr.responseText)
				if(data.errno && data.errno === 20521) {
          console.log(`${index + 1}/${length}`, uid, '关注失败')
        }
			}
		}
		xhr.open('POST', `/api/friendships/create?__rnd=${Date.now()}`)
    xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')
    // st
		xhr.send(`uid=${uid}&st=905aef&_spr=screen%3A375x812`)
	}
	let scheduled = setInterval(() => {
		index += 1
		follow(list[index])
	}, 2000)
	setTimeout(() => {
		clearTimeout(scheduled)
	}, 2000 * length)
})([/*上一步的结果 注意每天不要超过 150个 如果人很多你就分多天来关注 */])
```
