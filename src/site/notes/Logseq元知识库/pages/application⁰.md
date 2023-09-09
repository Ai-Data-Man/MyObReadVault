---
{"Aliases":["#Spring/Bean/作用域/application","#application⁰"],"dg-publish":true,"permalink":"/Logseq元知识库/pages/application⁰/","dgPassFrontmatter":true}
---

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
