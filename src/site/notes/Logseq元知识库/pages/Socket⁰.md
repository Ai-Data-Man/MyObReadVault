---
{"Aliases":["#Socket⁰"],"dg-publish":true,"permalink":"/Logseq元知识库/pages/Socket⁰/","dgPassFrontmatter":true}
---

Socket是计算机网络中的一个软件结构，用作在网络节点之间发送和接收数据的端点。它是通过网络架构的应用程序编程接口（API）定义的，其结构和属性由该接口定义。Socket只在应用程序运行期间的进程生命周期中创建。

由于TCP/IP协议在互联网的发展中的标准化，网络中最常用的术语是网络套接字（network socket），因此通常也称为Internet套接字。在这个语境下，套接字通过其套接字地址（socket address）对其他主机进行外部标识，套接字地址是由传输协议、IP地址和端口号构成的三元组。

套接字还用于表示节点内部进程间通信（IPC）的软件端点，它通常使用与网络套接字相同的API。

参考资料：
[1] Network socket. (2022, July 16). Retrieved from https://en.wikipedia.org/wiki/Network_socket
>
``` dataviewjs

let curPage = dv.current()
let curPageLink = "[[" + curPage.file.name + "]]"
let linkPages = curPage.file.inlinks.map(pl => dv.page(pl)).sort(lp => lp.ctime)
let textAndBlockObjMap = {}

function map2BlockObj(block) {
	if (textAndBlockObjMap[block.text]) {
		return textAndBlockObjMap[block.text]
	}
	let blockObj = {
		"text": block.text,
		"children": new Set(),
		"link": "<mark style=\"background: #BBFABBA6;\">" + block.link + "<\/mark>"
	}
	textAndBlockObjMap[block.text] = blockObj
	return blockObj
}

function iter(block) {
	var stack = []
	let blockObj = map2BlockObj(block)
	blockObj["level"] = 0 
	blockObj["source"] = block
	stack.push(blockObj)
	while(stack.length > 0) {
		let cur_block_obj = stack.shift()
		//dv.paragraph(cur_block_obj)
		let cur_block = cur_block_obj["source"]
		cur_block_obj["symbol"] = "▬ ".repeat(cur_block_obj["level"])
		if (cur_block_obj["level"] > 0 
			&& (cur_block.text.match(/^\d*[&|]?\[\[(?!.*(\.jpg|\.png)\]\]).*\]\]$/g))) {
			//dv.paragraph(cur_block_obj)
			continue
		}
		
		let childBlocks = cur_block.children
		for (let childBlock of childBlocks) {
			let childBlockObj = map2BlockObj(childBlock)
			childBlockObj["source"] = childBlock
			childBlockObj["level"] = cur_block_obj["level"] + 1
			if (childBlockObj["text"] != curPageLink) {
				cur_block_obj.children.add(childBlockObj)
			}
			stack.push(childBlockObj)
		}
	}
	
}

function rep(blockObj) {
	var stack = []
	stack.push(blockObj)
	while(stack.length > 0) {
		let cur_block_obj = stack.pop()
		dv.paragraph(cur_block_obj["link"] + cur_block_obj["symbol"] + cur_block_obj["text"])
		let childBlockObjs = cur_block_obj["children"]
		for (let childBlockObj of Array.from(childBlockObjs).reverse()) {
		    //dv.paragraph(childBlockObj["text"] + childBlockObj["symbol"])
			stack.push(childBlockObj)
		}
	} 
}


for (let page of linkPages) {
	//dv.paragraph(page)
	let blocks = page.file.lists
	for(let block of blocks) {
		let blockText = block.text
		//dv.paragraph(blockText)
		let partBlocks = blockText.split(/\d*[&|]/)
		//dv.paragraph(partBlocks)
		if (partBlocks.includes(curPageLink)) {
			//dv.paragraph(blockText)
			iter(block)
		}
	}
}


for (let k of Object.keys(textAndBlockObjMap).sort()) {
	o = textAndBlockObjMap[k]
	if (o["level"] == 1) {
		rep(o)
	}

}


```		
>

