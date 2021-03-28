# allcode
import os
import keep_alive
import discord
from datetime import datetime
from discord.ext import tasks
from pytz import timezone
import sqlite3
import re

dbname = 'bot.db'

conn = sqlite3.connect(dbname)
c = conn.cursor()  #DBへの接続

# 自分のBotのアクセストークン
TOKEN = 'ODE0ODk5NzI3NzkwMjQzODY1.YDkkgw.hFCrUl4vWnKwyyEVNMF_IttqP8A'

# 接続に必要なオブジェクトを生成
client = discord.Client()

member_mention = 0
count = 0
usechannel = 0
Schannel = 0
TimeCheck = 0


# 起動時に動作する処理
@client.event
async def on_ready():
	print('ready')


# メッセージ受信時に動作する処理
@client.event
async def on_message(message):
  global member_mention
  global usechannel
  usechannel = message.channel.id
  abridgement = message.content[:5]
  abridgement2 = message.content[11:16]
  abridgement3 = message.content[-18:0]
  if message.author.bot:
  	return
  if message.content == '/r':
		# table作成
  	c.execute(
		    "CREATE TABLE IF NOT EXISTS '%s' (id value,time value,event verchar)"
		    % usechannel)
  	conn.commit()
  	channel = client.get_channel(usechannel)
  	await channel.send(
		    "このチャンネルを登録しました。 https://cdn.discordapp.com/attachments/822840675312861184/822840968725659648/setumeisyo.png"
		)
  	return
  c.execute(
	    'SELECT COUNT(*) FROM sqlite_master WHERE TYPE="table" AND NAME= ? ',
	    (usechannel, ))
  if c.fetchone() == (0, ):  #存在しないとき
  	channel = client.get_channel(usechannel)
  	return
  else:
  	if message.content == '/delchannel':
  		c.execute("DROP TABLE IF EXISTS '%s'" % usechannel)
  		conn.commit()
  		channel = client.get_channel(usechannel)
  		await channel.send("このチャンネルを削除しました。")
  	else:
  		member_mention = (message.author.id)
  		if re.match(r'([0-1]?[0-9]|2[0-3]):([0-5]?[0-9])',
  		            abridgement2) != None:
  			if len(message.content) <= 16:
  				channel = client.get_channel(usechannel)
  				await channel.send("<@" + str(member_mention) + ">\n" +
					                   "時間の後にイベント名を入力してください。")
  			else:
  				Etime = message.content[0:16]
  				result1 = c.execute(
  				    "SELECT id time FROM `%s` Where id = ? AND time = ?" %
  				    usechannel, (member_mention, Etime))
  				if len(result1.fetchall()) != 0:
            channel = client.get_channel(usechannel)
            await channel.send("予定が重複しています。")
          else:
					  c.execute(
						    'insert into `%s` (id, time, event) values (?,?,?)'
						    % usechannel,
						    (member_mention, message.content[0:16],
						     message.content[17:]))
						conn.commit()
						c.execute('select * from `%s`' % usechannel)
						result = c.fetchall()
						print(result)
						channel = client.get_channel(usechannel)
						await channel.send("<@" + str(member_mention) + ">\n" +
						                   message.content[0:16] + "に" +
						                   message.content[17:] +
						                   "のイベントを設定しました。")
			elif re.match(r'([0-1]?[0-9]|2[0-3]):([0-5]?[0-9])',
			              abridgement) != None:
				now = datetime.now(timezone('Asia/Tokyo')).strftime('%Y/%m/%d')
				c.execute(
				    'insert into `%s` (id, time, event) values (?,?,?)' %
				    usechannel,
				    (member_mention, now + ' ' + message.content[0:5],
				     message.content[6:]))
				c.execute('select * from `%s`' % usechannel)
				result = c.fetchall()
				print(result)
				channel = client.get_channel(usechannel)
				await channel.send("<@" + str(member_mention) + ">\n" + now +
				                   ' ' + message.content[0:5] + "に" +
				                   message.content[6:] + "のイベントを設定しました。")
      elif re.match(r'([0-9])',abridgment) != None:
        c.execute(
						    'insert into `%s` (id, time, event) values (?,?,?)'
						    % usechannel,
						    (member_mention, message.content[0:16],
						     message.content[17:]))
				conn.commit()
				c.execute('select * from `%s`' % usechannel)
				result = c.fetchall()
				print(result)
				channel = client.get_channel(usechannel)
				await channel.send("<@" + str(member_mention) + ">\n" + message.content[0:16] + "に" + message.content[17:] +"のイベントを設定しました。")
			elif message.content == '/all':
				channel = client.get_channel(usechannel)
				c.execute(
				    'select time, event from `%s` where id = ?' % usechannel,
				    (member_mention, ))
				AllResult = sorted(c.fetchall())
				print(AllResult)
				number = len(AllResult)
				if number != 0:
					number2 = 0
					await channel.send("<@" + str(member_mention) + ">" +
					                   "のイベント一覧\n ")
					for num in range(number):
						await channel.send(AllResult[number2][0] + " " +
						                   AllResult[number2][1])
						number2 += 1
				else:
					await channel.send("<@" + str(member_mention) +
					                   ">さんの登録しているイベントはありません。")
			elif message.content == '/delall':
				c.execute('DELETE FROM `%s` WHERE id = ?' % usechannel,
				          (member_mention, ))
				channel = client.get_channel(usechannel)
				conn.commit()
				await channel.send("<@" + str(member_mention) + ">" +
				                   "さんの登録しているすべてのイベントを削除しました。")
			elif message.content == '/del':
				now = datetime.now(
				    timezone('Asia/Tokyo')).strftime('%Y/%m/%d %H:%M')
				c.execute('DELETE FROM `%s` WHERE time < ?' % usechannel,
				          (now, ))
				channel = client.get_channel(usechannel)
				conn.commit()
				await channel.send("<@" + str(member_mention) + ">" +
				                   "さんの登録している過去のイベントを削除しました。")
			elif message.content == '/c':
				channel = client.get_channel(usechannel)
				await channel.send("<@" + str(member_mention) + ">" + "\n"
				                   '/r : チャンネルにbotを導入 \n' +
				                   '/delchannel : チャンネルでbotを使えないようにする \n' +
				                   '/all : 現在登録しているすべてのイベントを表示 \n' +
				                   '/delall : 現在登録しているすべてのイベントを削除　\n' +
				                   '/del : 現在時刻より過去のイベントを削除')
				await channel.send(
				    'https://cdn.discordapp.com/attachments/822840675312861184/822840968725659648/setumeisyo.png'
				)
			else:
				channel = client.get_channel(usechannel)
				await channel.send('型は[YYYY/MM/DD hh:mm イベント名]で送信してください。')


#編集されたときに走る処理
@client.event
async def on_message_edit(befor, after):
	delTime = befor.content[0:16]
	delEvent = befor.content[17:]
	delID = befor.author.id
	afterID = after.author.id
	usechannel = after.channel.id
	channel = client.get_channel(usechannel)
	c.execute(
	    'SELECT * FROM `%s` WHERE id = ? AND time = ? AND event = ?' %
	    usechannel, (delID, delTime, delEvent))
	result2 = len(c.fetchall())
	print(result2)
	if result2 == 0:
		c.execute(
		    'insert into `%s` (id, time, event) values (?,?,?)' % usechannel,
		    (afterID, after.content[0:16], after.content[17:]))
		conn.commit()
		await channel.send("<@" + str(afterID) + ">\n" + after.content[0:16] +
		                   "に" + after.content[17:] + "のイベントを設定しました。")
	else:
		c.execute(
		    'DELETE FROM `%s` WHERE id = ? AND time = ? AND event = ?' %
		    usechannel, (delID, delTime, delEvent))
		conn.commit()
		c.execute(
		    'insert into `%s` (id, time, event) values (?,?,?)' % usechannel,
		    (afterID, after.content[0:16], after.content[17:]))
		conn.commit()
		await channel.send("<@" + str(afterID) + ">" + "\n" +
		                   befor.content[0:16] + "の" + befor.content[17:] +
		                   "のイベントを削除し、" + after.content[0:16] + "の" +
		                   after.content[17:] + 'のイベントを追加しました。')


#リアクションを付けられたときに走る処理
@client.event
async def on_reaction_add(reaction, user):
	if reaction.message.content.startswith('202'):
		delTime = reaction.message.content[0:16]
		delEvent = reaction.message.content[17:]
		delID = user.id
		channel = client.get_channel(usechannel)
		c.execute(
		    'SELECT * FROM `%s` WHERE id = ? AND time = ? AND event = ?' %
		    usechannel, (delID, delTime, delEvent))
		result2 = len(c.fetchall())
		print(result2)
		c.execute(
		    'DELETE FROM `%s` WHERE id = ? AND time = ? AND event = ?' %
		    usechannel, (delID, delTime, delEvent))
		conn.commit()
		if result2 == 0:
			await channel.send("<@" + str(user.id) +
			                   ">\n 自分以外のイベントを削除することはできません。")
		else:
			await channel.send("<@" + str(user.id) + ">" + "\n" +
			                   reaction.message.content[0:16] + "の" +
			                   reaction.message.content[17:] + "のイベントを削除しました。")


# 〇〇秒に一回ループ
@tasks.loop(seconds=60)
async def time_check():
	global TimeCheck
	c.execute('SELECT name FROM sqlite_master WHERE TYPE="table"')
	tablecount = len(c.fetchall())
	c.execute('SELECT name FROM sqlite_master WHERE TYPE="table"')
	tables = c.fetchall()
	print(tablecount)
	print(tables)

	# 現在の時刻
	now = datetime.now(timezone('Asia/Tokyo')).strftime('%Y/%m/%d %H:%M')
	#起動時間
	print('起動時間:' + str(TimeCheck))
	TimeCheck += 1
	for num in range(tablecount):
		c.execute('SELECT time FROM `%s` WHERE time = ?' % tables[num][0],
		          (now, ))
		result2 = str(c.fetchall())
		#print(str(num) + '回目')
		#print(result2)
		if result2 != '[]':
			SendChannel = tables[num][0]
			await SendMessage(now, SendChannel)


#ループ処理
time_check.start()


# 指定時間に走る処理
async def SendMessage(timenow, Schannel):
	c.execute('SELECT * FROM `%s` WHERE time = ?' % Schannel, (timenow, ))
	EventResult = c.fetchall()
	print(EventResult)
	channel = client.get_channel(int(Schannel))
	print(Schannel)
	await channel.send("<@" + str(EventResult[0][0]) + "> さん" +
	                   EventResult[0][2] + "の時間です。")


# Botの起動とDiscordサーバーへの接続
keep_alive.keep_alive()
client.run(os.getenv('TOKEN'))
conn.close()
