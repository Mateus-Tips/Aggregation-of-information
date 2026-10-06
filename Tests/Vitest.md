`vi.resetModules()`

Limpa o cache de imports antes de cada teste, fazendo o módulo ser reimportado do zero incluindo o estado inicial.

O que mockResolvedValue faz
Quando você tem um vi.fn(), ele não faz nada por padrão. O mockResolvedValue diz: "quando essa função for chamada, retorna essa Promise resolvida com esse valor".