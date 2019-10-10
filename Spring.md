1. @Autowired注解：默认是按照类型装配的。

	    @Service  
		public class UserService {  
		  public void add(){  
		        System.out.println("UserService add...");  
		        userRepository.save();  
		    }  
		}
	
		@Controller  
		public class UserController {  
		  
		  @Autowired  
		  private UserService service;  
		  
		  public void execute(){  
		     System.out.println("UserController execute...");  
		     service.add();  
		  }  
		}
	  如上例，按照这种方式进行注入，UserService是可以成功注入的，这就说明@Autowired是按照类型注入，而非name。
2. 是否允许为null：如果在进行注入时，允许为null，在这种情况下可以通过@Autowired(required = false)来指定。默认情况下required=true
3. 当依赖的类型出现多个时候：

		public interface UserRepository {  
		    void save();  
		}
		
		@Repository  
		public class JdbcRepositoryImpl implements UserRepository {  
		  
		  @Override  
		  public void save() {  
		     System.out.println("jdbc repository saved...");  
		  }  
		}
		
		@Repository("userRepository")  
		public class UserRepositoryImpl implements UserRepository {  
		  
		  @Override  
		  public void save() {  
		     System.out.println("UserRepositoryImpl save...");  
		  }  
		}
	如这里的UserRepository有两个实现类时，然后需要在UserService类中使用时：

		@Service  
		public class UserService {  
		  
		 @Autowired  
		 private UserRepository userRepository;  
		 
		  public void add(){  
		    System.out.println("UserService add...");  
		    userRepository.save();  
		  }  
		}
	按照上面这种写法，如果按照第1条中的准则，默认按照类型进行装配，则在这里就出现了歧义，因为UserRepository有两个实现类：UserRepositoryImpl 和JdbcRepositoryImpl ，这也就意味着按照默认的规则已经不够使了。所以有了第二条规则：可以按照name来进行装配。如上例，这里使用的就是UserRepositoryImpl （因为bean的名字默认就是类名首字母小写）。换言之，这里还可以写成：

		@Autowired  
		private UserRepository jdbcRepositoryImpl;
	那么此时注入的就是JdbcRepositoryImpl。
4. @Qualifier：除了第3条中阐明的方式之外，我们还可以借助于@Qualifier注解，
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTcyMTUwNzMxMSwxMjkwMDI0MDg1LC0yMD
g4NzQ2NjEyXX0=
-->