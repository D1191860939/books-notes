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
	   如上例，  

<!--stackedit_data:
eyJoaXN0b3J5IjpbMzk3MTk4NTMwLDEyOTAwMjQwODUsLTIwOD
g3NDY2MTJdfQ==
-->