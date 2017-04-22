# SOLID

O SOLID é um acrônimo criado por criado por Michael Feathers a partir dos cinco primeiros princípiocs ("five first principles") da programação orientada à objetos identificados por Robert C. Martin no início dos anos 2000.

Esses princípios tem como objetivo obter as vantagens da orientação a objetos através de um código de alta qualidade e permita:
- Leitura, testes e manutenção fáceis
- Extensibilidade com o menor esforço possível
- Reaproveitamento
- Maximização do tempo de utilização do código

A aplicação correta dos princípios SOLID permite um código altamente estruturado e desacoplado de modo que são evitadas, devido à uma estrutura frágil:
- Dificuldades no teste
- Dificuldades no isolamento de funcionalidades
- Duplicação de código
- Quebra de código em vários lugares após alterações

A seguir vamos estudar cada um dos 5 príncipios que compõem o SOLID.

[Single Responsability Principle](#single-responsability-principle)

[Open-Closed Principle](#open-closed-principle)

[Liskov Substitution Principle](#liskov-substitution-principle)

[Interface Segregation Principle](#interface-segregation-principle)

[Dependency Inversion Principle](#dependency-inversion-principle)

## Single Responsability Principle

> "A class should have one, and only one, reason to change"

O Princípio da Responsabilidade Única (Single Responsability Principle) diz que "uma classe deve ter um único motivo para ser modificada". Em outras palavras, uma classe deve ter uma única responsabilidade, um único motivo de existir.

Veja o exemplo da classe Usuário abaixo.

### Violação do SRP

```c#
public class Usuario
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public string Email { get; set; }
    public string CPF { get; set; }

    public Tuple<bool, string> InserirUsuario()
    {
        if (CPF.Length != 11)
            return new Tuple<bool, string>(false, "O CPF deve ter 11 caracteres");

        using (var connection = new SqlConnection())
        {
            var command = new SqlCommand();

            connection.ConnectionString = "Local";

            command.Connection = connection;
            command.CommandType = System.Data.CommandType.Text;
            command.CommandText = $"INSERT INTO USUARIO (ID, NOME, EMAIL, CPF) VALUES (@id, @nome, @email, @cpf)";

            command.Parameters.AddWithValue("id", Id);
            command.Parameters.AddWithValue("nome", Nome);
            command.Parameters.AddWithValue("email", Email);
            command.Parameters.AddWithValue("cpf", CPF);

            connection.Open();
            command.ExecuteNonQuery();
        }

        return new Tuple<bool, string>(true, "Usuário inserido com sucesso");
    }
}
```

Da maneira como foi implantada, essa classe viola o SRP uma vez que uma classe não deve delegar responsabilidades à ela própria. No caso do exemplo, a classe não deve validar a si mesma ou se adicionar ao banco. Outro exemplo de violação está no fato de que um método de Inserção de um determinado registro não deve ter a responsabilidade de validar esse registro.

Logo, podemos concluir que uma classe construída da maneira demonstrada acima possui múltiplas responsabilidades, o que vai contra o SRP.

### SRP da maneira correta

Para apicar o SRP de uma maneira correta então, devemos separar as responsabilidades descritas acima, de modo que haja exclusivamente uma classe para cada responsabilidade:

- Manter as propriedades e indicar se estas são válidas ou não.

```c#
public class Usuario
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public string Email { get; set; }
    public string CPF { get; set; }
    public bool IsValid { get { return UsuarioValidation.Validar(); } }
}
```

- Executar essa validação.

```c#
public static class UsuarioValidation
{
    public bool Validar(Usuario usuario)
    {
        return usuario != null && usuario.CPF != null && usuario.CPF != String.Empty && usuario.CPF.Length == 11;
    }
}
```

- Interagir com o Banco.

```c#
public class UsuarioRepository
{
    public bool Adicionar(Usuario usuario)
    {
        try
        {
            using (var connection = new SqlConnection())
            {
                var command = new SqlCommand();

                connection.ConnectionString = "Local";
                command.Connection = connection;
                command.CommandType = System.Data.CommandType.Text;
                command.CommandText = $"INSERT INTO USUARIO (ID, NOME, EMAIL, CPF) VALUES (@id, @nome, @email, @cpf)";

                command.Parameters.AddWithValue("id", Id);
                command.Parameters.AddWithValue("nome", Nome);
                command.Parameters.AddWithValue("email", Email);
                command.Parameters.AddWithValue("cpf", CPF);

                connection.Open();
                command.ExecuteNonQuery();
            }

            return true;
        }
        catch (Exception)
        {
            return false;
        }
    }
}
```

- Executar o processo de adição de um novo Usuário.
```c#
public class UsuarioService
{
    public bool Inserir(Usuario usuario)
    {
        if (usuario.IsValid())
            return false;

        var repository = new UsuarioRepository();
        repository.Adicionar(usuario);
    }
}
```

De uma maneira bem grosseira, corrigimos a violação do SRP separando uma única classe Usuário, que antes era responsável por verificar se suas propriedades eram válidas, criar uma conexão com o banco e efetuar a inserção, em 4 classes com responsabilidades únicas. Dessa forma, desacoplamos o código, facilitando alterações e testes do mesmo. 

Imagine, por exemplo, que os seus dados hoje guardados em um banco SQL passem a ser guardados em um arquivo XML. A única alteração a ser feita será no Repositório, mantendo todo o resto do nosso código intacto, uma vez que o Serviço e a Validação não precisam saber de que maneira é feita a conexão e a inserção na base de dados.

## Open-Closed Principle

## Liskov Substitution Principle

## Interface Segregation Principle

## Dependency Inversion Principle
