---
{"dg-publish":true,"permalink":"/Logseq元问题库/pages/？？Netty代码示例/","dgPassFrontmatter":true}
---

```java
		// 创建bossGroup和workerGroup 
			// 说明
			// 1. 使用NioEventLoopGroup创建两个线程组 bossGroup 和 workerGroup
			// 2. NioEventLoopGroup构造器传入的参数是线程数，默认是cpu核数*2
			// 3. bossGroup 只是处理连接请求 , 真正的和客户端业务处理 , 会交给 workerGroup完成。只需要1个线程即可，因此bossGroup的线程数设置为1
			// 4. workerGroup 真正的和客户端业务处理 , 会交给 workerGroup完成。因此workerGroup的线程数要设置的不止1个，一般保持默认即可，即cpu核数*2，也可按需调整
			EventLoopGroup bossGroup = new NioEventLoopGroup(1);
			EventLoopGroup workerGroup = new NioEventLoopGroup(8);
	
			try {
				// 创建服务器端的启动对象，配置参数
				ServerBootstrap bootstrap = new ServerBootstrap();
	
				// 使用链式编程来进行设置
				bootstrap.group(bossGroup, workerGroup) // 设置两个线程组
						.channel(NioServerSocketChannel.class) // 使用 NioServerSocketChannel 作为服务器的通道实现
						.option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列得到连接个数
						.childOption(ChannelOption.SO_KEEPALIVE, true) // 设置保持活动连接状态
						.childHandler(new ChannelInitializer<SocketChannel>() {// 创建一个通道测试对象(匿名对象)
							// 给 pipeline 设置处理器
							@Override
							protected void initChannel(SocketChannel ch) throws Exception {
								ch.pipeline().addLast(new NettyServerHandler());
							}
						}); // 给我们的 workerGroup 的 EventLoop 对应的管道设置处理器
	
				System.out.println("......服务器 is ready...");
	
				// bootstrap.bind()启动服务器(并绑定端口)，bind是异步操作，所以需要调用sync()方法来等待异步操作执行完毕以保证服务器初始化完成
				ChannelFuture cf = bootstrap.bind(6668).sync();
	
				// 给 cf 注册监听器，监控我们关心的事件
				cf.addListener(new ChannelFutureListener() {
					@Override
					public void operationComplete(ChannelFuture future) throws Exception {
						if (cf.isSuccess()) { //如果监听端口成功
							System.out.println("监听端口 6668 成功");
						} else {
							System.out.println("监听端口 6668 失败");
						}
					}
				});
	
				// cf.channel.closeFuture()是异步操作，它直接返回一个代表“未来通道被关闭”的ChannelFuture异步结果，所以需要调用其sync()方法来等待通道关闭事件的发生，这样只要通道不关闭，服务器就会一直监听6668端口，不会退出；否则，因为下面的代码将直接执行完成，会进入finally代码块，导致服务器退出。
				cf.channel().closeFuture().sync();
	
			} finally {
				// 优雅地关闭服务器
				bossGroup.shutdownGracefully();
				workerGroup.shutdownGracefully();
			}
	
	
```


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
			&& (cur_block.text.match(/\d*[&|]?\[\[(?!.*(\.jpg|\.png)\]\]).*\]\]/g))) {
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
