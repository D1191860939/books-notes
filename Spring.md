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
2. 是否允许为null：	   

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNDY0ODI3NTEsMTI5MDAyNDA4NSwtMj
A4ODc0NjYxMl19
-->