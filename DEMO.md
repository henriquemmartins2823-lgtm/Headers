Demo - Headers
Resumo do projeto:

O Headers é uma ferramenta que faz uma avaliação dos cabeçalhos de segurança HTTP de URLs. Ela usa httpx para fazer as requisições, Rich para montar relatórios no terminal e argparse - a biblioteca que lê os argumentos digitados no terminal - para interpretar as opções da CLI (interface de linha de comando, para interpretar as opções digitadas no terminal). O fluxo principal é: scan() busca a resposta, evaluate_header() classifica cada regra como ok, weak ou missing, e ScanReport calcula a pontuação de 0 a 100 e a nota de A a F. O scanner só lê respostas de URLs autorizadas; ele não explora vulnerabilidades nem modifica nenhum site.

No desafio 1, adicionei o sétimo cabeçalho, Cross-Origin-Opener-Policy, para a lista RULES. Para adicionar esse sétimo cabeçalho, criei uma nova HeaderRule com severidade alta e com o valor sendo obrigatoriamente como same-origin, porque apenas a presença do header não adianta se ele vier configurado errado. Essa configuração ajuda a reduzir interações perigosas entre origens e riscos de XS-Leaks. Escolhi manter a regra na mesma lista das outras porque ao acrescentar uma regra, ela entra automaticamente no scan, na tabela e na pontuação. Além disso, também atualizei os testes e o mock de resposta completa para incluir o novo header, em vez de mudar os pesos da rubrica original. Agora que existem três headers de gravidade alta, dois médios e dois baixos, o total bruto passou a ser 130 pontos, mas a nota continua de 0 a 100 porque o cálculo usa a proporção entre os pontos obtidos e os pontos possíveis.

No desafio 2, adicionei a flag --json com argparse. Quando essa flag é usada, o programa transforma cada ScanReport em um dicionário com asdict(), adiciona também a pontuação e a nota calculadas e usa json.dumps() para gerar uma saída organizada. A escolha do JSON permite que outro script, uma pipeline ou um dashboard consumam os resultados sem precisar interpretar uma tabela colorida. Depois do desafio 4, adaptei esse modo para juntar os relatórios em uma única lista JSON, em vez de imprimir vários JSONs separados.

No desafio 3, coloquei a flag --verbose. Para isso, scan() passou a guardar todos os cabeçalhos que foram recebidos em all_headers, dentro de ScanReport, e o renderizador só mostra eles quando o usuário pede --verbose. Assim, a saída continua curta e focada nos sete headers de segurança, enquanto o com --verbose ajuda a visualizar uma avaliação ou conferir a resposta completa do servidor. Usei field(default_factory=dict) em all_headers para que os relatórios que foram criados nos testes continuem válidos mesmo sem uma lista de cabeçalhos fornecida.

No desafio 4, mudei o argumento url para nargs="+", fazendo com que args.url se torne uma lista. Diante disso, em main(), criei um loop que escaneia cada URL, guarda os relatórios que foram bem-sucedidos, monta a tabela-resumo final e acumula os dados para o modo JSON. Também fiz com que as falhas de rede não interrompessem as outras URLs e guardei o pior código de saída com max(): a nota A/B retornam 0, C/D retornam 1 e F ou o erro de rede retornam 2.

A lógica de avaliação está separada da requisição de rede, então os testes usam respostas simuladas com respx e não dependem de nenhum site externos. Abaixo estão os resultados do just test, just lint e das URLs autorizadas usadas no vídeo, e também o link da gravação. Com esse projeto, consegui praticar requisições HTTP, headers de segurança, dataclasses (que eu não tinha aprendido no primeiro semestre no pygame!), testes, CLI e códigos de saída; como próximos passos, eu implementaria a flag de limite --allow-warnings (Permitir que o usuário diga "estou ok com a nota C, saia com erro apenas se a nota for D ou inferior".), cache (Quando o usuário escanear a mesma URL novamente dentro da última hora, retorne o resultado em cache em vez de acessar a rede) e detecção de HTTP/HTTPS mistos (Se o usuário passar uma URL http:// e o servidor redirecionar para https://, mencione isso com destaque na saída), visto que são mais simples de fazer do que as implementações dos desafios avançados.

https://youtu.be/6eD1fEB5TEE Vídeo de apresentação

collected 13 items                                                                                                                           

test_http_headers_scanner.py::test_evaluate_header_present_with_required_substring PASSED                                              [  7%]
test_http_headers_scanner.py::test_evaluate_header_present_without_required_substring PASSED                                           [ 15%]
test_http_headers_scanner.py::test_evaluate_header_missing PASSED                                                                      [ 23%]
test_http_headers_scanner.py::test_evaluate_header_hsts_max_age_zero_is_weak PASSED                                                    [ 30%]
test_http_headers_scanner.py::test_evaluate_header_case_insensitive_lookup PASSED                                                      [ 38%]
test_http_headers_scanner.py::test_evaluate_header_no_must_match_treats_presence_as_ok PASSED                                          [ 46%]
test_http_headers_scanner.py::test_score_all_ok_is_100 PASSED                                                                          [ 53%]
test_http_headers_scanner.py::test_score_all_missing_is_zero PASSED                                                                    [ 61%]
test_http_headers_scanner.py::test_grade_threshold_a_at_90_percent PASSED                                                              [ 69%]
test_http_headers_scanner.py::test_grade_threshold_b_at_83_percent PASSED                                                              [ 76%]
test_http_headers_scanner.py::test_scan_mocks_a_clean_response_and_grades_it_correctly PASSED                                          [ 84%]
test_http_headers_scanner.py::test_scan_flags_missing_and_weak_headers PASSED                                                          [ 92%]
test_http_headers_scanner.py::test_scan_records_final_url_after_redirect PASSED                                                        [100%]


$ just lint
=== Ruff ===
uv run ruff check http_headers_scanner.py test_http_headers_scanner.py
All checks passed!

=== Pylint ===
uv run pylint http_headers_scanner.py

-------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 9.93/10, +0.07)
