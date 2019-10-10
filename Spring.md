1. @Autowired注解：默认是按照类型装配的。

    @Service  
	public class UserService {  
	  
	    @Autowired  
	//    @Qualifier(value = "jdbcRepositoryImpl")  
	  private UserRepository userRepository;  
	  
	//    public UserService(UserRepository userRepository) {  
	//        this.userRepository = userRepository;  
	//    }  
	  
	  public void add(){  
	        System.out.println("UserService add...");  
	        userRepository.save();  
	    }  
	}
    

<!--stackedit_data:
eyJoaXN0b3J5IjpbNzcyNTIyODM5LDEyOTAwMjQwODUsLTIwOD
g3NDY2MTJdfQ==
-->