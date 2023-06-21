# biblioteca-room-kotlin
 exemplo de código em Kotlin que ilustra como usar a biblioteca Room para armazenar e exibir dados de jogos da seleção brasileira na Copa:

1. Crie uma classe de entidade para representar um jogo:

```kotlin
@Entity(tableName = "games")
data class Game(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val date: Long = 0,
    val opponent: String = "",
    val score: String = ""
)
```

2. Crie uma interface DAO (Data Access Object) para acessar os dados de jogos:

```kotlin
@Dao
interface GameDao {
    @Query("SELECT * FROM games")
    fun getAll(): LiveData<List<Game>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(game: Game)
}
```

3. Crie um banco de dados usando a classe abstrata RoomDatabase:

```kotlin
@Database(entities = [Game::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun gameDao(): GameDao

    companion object {
        private const val DATABASE_NAME = "app_database"

        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    DATABASE_NAME
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

4. Na classe de atividade ou fragmento que exibe a lista de jogos, crie uma instância do GameViewModel e observe os dados de jogos usando o LiveData:

```kotlin
class GameListFragment : Fragment() {
    private lateinit var viewModel: GameViewModel

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val binding = FragmentGameListBinding.inflate(inflater, container, false)

        val adapter = GameAdapter()
        binding.gameList.adapter = adapter

        viewModel = ViewModelProvider(this).get(GameViewModel::class.java)
        viewModel.games.observe(viewLifecycleOwner, { games ->
            adapter.submitList(games)
        })

        return binding.root
    }
}
```

5. Na classe de ViewModel, injete uma instância do GameDao e use-a para inserir e recuperar dados de jogos:

```kotlin
class GameViewModel(application: Application) : AndroidViewModel(application) {
    private val gameDao = AppDatabase.getInstance(application).gameDao()
    val games: LiveData<List<Game>> = gameDao.getAll()

    fun insert(game: Game) {
        viewModelScope.launch {
            gameDao.insert(game)
        }
    }
}
```

Espero que este exemplo lhe dê uma ideia de como usar a biblioteca Room para armazenar e exibir dados de jogos em seu aplicativo Android. Lembre-se de que este é apenas um exemplo e que você precisará personalizar e adaptar o código para atender às suas necessidades específicas.
