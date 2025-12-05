# Best Practices for PrimeFaces, JoinFaces & Java Development

Project Structure & Naming Conventions

# Recommended Folder Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── empresa/
│   │           └── proyecto/
│   │               ├── config/          # Configuration classes
│   │               ├── controller/      # Backing beans (@Named)
│   │               ├── model/           # Entity classes (JPA)
│   │               ├── repository/      # Data access layer (Spring Data JPA)
│   │               ├── service/         # Business logic layer
│   │               ├── dto/             # Data Transfer Objects
│   │               ├── converter/       # JSF converters
│   │               ├── validator/       # Custom validators
│   │               ├── util/            # Utility classes
│   │               └── exception/       # Custom exceptions
│   ├── resources/
│   │   ├── META-INF/
│   │   │   └── resources/
│   │   │       └── empresa/            # Web resources
│   │   │           ├── css/
│   │   │           ├── js/
│   │   │           ├── images/
│   │   │           └── layout/         # Templates
│   └── webapp/
│       └── WEB-INF/
│           ├── faces-config.xml
│           ├── web.xml
│           └── templates/              # Facelets templates
└── test/
```


# File Naming Conventions
```
✅ CORRECTO:
- UsuarioController.java
- UsuarioService.java
- UsuarioRepository.java
- Usuario.java (Entity)
- UsuarioDTO.java
- usuario-form.xhtml
- usuario-list.xhtml
- usuario-detail.xhtml

❌ EVITAR:
- userController.java (inconsistente)
- Usuario_Form.xhtml (guiones bajos)
- formUsuario.xhtml (inverso)
```

# Naming Conventions (Bilingual Team - English Recommended)

Java Classes & Methods

```
// ✅ Controller/Backing Beans
@Named
@ViewScoped
public class UserController {
    private UserDTO selectedUser;
    private List<UserDTO> userList;
    private String searchFilter;
    
    // ✅ Methods - action/event handlers
    public void loadUsers() { }
    public String saveUser() { }
    public void deleteUser() { }
    public void onRowSelect(SelectEvent<UserDTO> event) { }
    
    // ✅ Boolean methods
    public boolean isUserActive() { }
    public boolean hasPermissions() { }
    
    // ✅ Getter/Setter patterns
    public UserDTO getSelectedUser() { }
    public void setSelectedUser(UserDTO selectedUser) { }
}

// ✅ Service Layer
@Service
public class UserService {
    public UserDTO createUser(UserDTO userDto) { }
    public Page<UserDTO> findUsers(UserFilter filter, Pageable pageable) { }
    public void validateUserData(UserDTO userDto) { }
}

// ✅ Repository Layer
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByStatus(Status status);
}
```
# Variable Naming

```
// ✅ Collections
private List<UserDTO> userList;          // List of users
private Set<Role> roleSet;               // Set of roles
private Map<String, Permission> permissionMap; // Map permissions
private List<UserDTO> filteredUsers;     // Filtered results
private List<SelectItem> countryOptions; // Options for select

// ✅ Individual objects
private UserDTO selectedUser;
private UserDTO newUser;
private UserDTO editedUser;

// ✅ Strings & Primitives
private String searchText;
private String emailAddress;
private String phoneNumber;
private Integer pageSize;
private Boolean isActive;

// ✅ Flags & states
private boolean isEditMode;
private boolean isInitialized;
private boolean hasErrors;
```
# Archivos XHTML:
```
✅ CORRECTO:
- usuario-list.xhtml
- usuario-form.xhtml
- usuario-detail.xhtml
- usuario-search.xhtml

✅ Templates:
- template.xhtml
- crud-template.xhtml
- admin-template.xhtml

✅ Fragmentos:
- includes/form-buttons.xhtml
- includes/data-table.xhtml
- modals/delete-confirm.xhtml
```
# Formulario XHTML con Validaciones
```
<h:form id="userForm">
    <p:panel header="Registro de Usuario">
        <p:messages id="messages" />
        
        <p:panelGrid columns="2">
            <p:outputLabel for="nombre" value="Nombre *" />
            <p:inputText id="nombre" 
                         value="#{controller.entity.nombre}"
                         required="true"
                         requiredMessage="Nombre requerido">
                <f:validateLength minimum="3" maximum="100" />
            </p:inputText>
            <p:message for="nombre" />
            
            <p:outputLabel for="email" value="Email *" />
            <p:inputText id="email"
                         value="#{controller.entity.email}"
                         required="true">
                <f:validateRegex pattern="^[A-Za-z0-9+_.-]+@(.+)$" />
                <p:ajax event="blur" 
                        listener="#{controller.validateEmail}" />
            </p:inputText>
        </p:panelGrid>
        
        <p:commandButton value="Guardar"
                         action="#{controller.save}"
                         update="@form :globalGrowl"
                         process="@form"
                         icon="pi pi-save" />
    </p:panel>
</h:form>
```
# DataTable con Lazy Loading
```
<p:dataTable id="userTable"
             value="#{controller.lazyModel}"
             var="user"
             paginator="true"
             rows="10"
             lazy="true">
    
    <p:column headerText="ID" sortBy="#{user.id}">
        #{user.id}
    </p:column>
    
    <p:column headerText="Nombre" sortBy="#{user.nombre}">
        #{user.nombre}
    </p:column>
    
    <p:column headerText="Acciones">
        <p:commandButton icon="pi pi-pencil"
                         action="#{controller.prepareEdit(user)}"
                         title="Editar" />
    </p:column>
</p:dataTable>
```


# Scope Management - @ViewScoped vs Others
When to use @ViewScoped
```
// ✅ USAR @ViewScoped cuando:
@Named
@ViewScoped  // 👈 Perfecto para formularios complejos
public class UserFormController {
    private UserDTO user;
    private List<Role> availableRoles;
    
    @PostConstruct
    public void init() {
        // Mantiene estado durante interacciones con la misma vista
        // Sobrevive a AJAX requests dentro de la misma página
    }
    
    // Ejemplos ideales:
    // - Formularios con múltiples pasos
    // - CRUD con dialogs modales
    // - Tablas con paginación/filtros
    // - Wizard de varios pasos
}

// ✅ Ejemplo de formulario con @ViewScoped
@Named
@ViewScoped
public class RegistrationController {
    private UserDTO user = new UserDTO();
    private AddressDTO address = new AddressDTO();
    private List<DocumentType> documentTypes;
    private int currentStep = 1;
    
    @PostConstruct
    public void init() {
        documentTypes = documentService.getDocumentTypes();
    }
    
    public void nextStep() {
        currentStep++;
        // El estado se mantiene entre pasos
    }
    
    public void save() {
        userService.saveUser(user, address);
        FacesUtil.addSuccessMessage("Usuario registrado");
    }
}
```

# When NOT to use @ViewScoped
```
// ✅ USAR @RequestScoped cuando:
@Named
@RequestScoped  // 👈 Para operaciones simples
public class LoginController {
    private String username;
    private String password;
    
    public String login() {
        // Simple autenticación - no necesita mantener estado
        return "dashboard?faces-redirect=true";
    }
}

// ✅ USAR @SessionScoped cuando:
@Named
@SessionScoped  // 👈 Datos de sesión del usuario
public class UserSessionController {
    private UserDTO currentUser;
    private Locale userLocale;
    private Theme selectedTheme;
    
    // Información que persiste durante toda la sesión
}

// ✅ USAR @ApplicationScoped cuando:
@Named
@ApplicationScoped  // 👈 Datos globales de la aplicación
public class ApplicationConfigController {
    private List<SystemParameter> systemParameters;
    private Map<String, String> appSettings;
    
    @PostConstruct
    public void init() {
        // Carga datos compartidos por todos los usuarios
    }
}
```
# Form Implementation Best Practices

```
<!-- usuario-form.xhtml -->
<ui:composition template="/WEB-INF/templates/template.xhtml"
                xmlns="http://www.w3.org/1999/xhtml"
                xmlns:h="http://xmlns.jcp.org/jsf/html"
                xmlns:p="http://primefaces.org/ui"
                xmlns:f="http://xmlns.jcp.org/jsf/core"
                xmlns:ui="http://xmlns.jcp.org/jsf/facelets">

    <ui:define name="content">
        <h:form id="userForm">
            
            <!-- Messages Globales -->
            <p:messages id="messages" showDetail="true" closable="true"/>
            
            <!-- Panel con validación -->
            <p:panel header="Registro de Usuario" styleClass="form-panel">
                
                <!-- Grid responsivo -->
                <p:grid layout="grid" columns="2" columnClasses="col-6, col-6">
                    
                    <!-- Input con validación -->
                    <p:outputLabel for="email" value="Email *"/>
                    <p:inputText id="email" 
                                 value="#{userController.user.email}"
                                 required="true"
                                 requiredMessage="Email es requerido"
                                 validatorMessage="Formato inválido">
                        <f:validateRegex pattern="[^@]+@[^@]+\.[^@]+"/>
                        <p:ajax event="blur" 
                                update="@this emailMsg"/>
                    </p:inputText>
                    <p:message for="email" id="emailMsg"/>
                    
                    <!-- Select con opciones -->
                    <p:outputLabel for="role" value="Rol *"/>
                    <p:selectOneMenu id="role" 
                                     value="#{userController.user.roleId}"
                                     required="true">
                        <f:selectItem itemLabel="Seleccione..." noSelectionOption="true"/>
                        <f:selectItems value="#{userController.roleOptions}"
                                       var="role"
                                       itemLabel="#{role.name}"
                                       itemValue="#{role.id}"/>
                    </p:selectOneMenu>
                    
                    <!-- Date picker -->
                    <p:outputLabel for="birthDate" value="Fecha Nacimiento"/>
                    <p:calendar id="birthDate"
                                value="#{userController.user.birthDate}"
                                pattern="dd/MM/yyyy"
                                locale="es"/>
                    
                </p:grid>
                
                <!-- Botones de acción -->
                <p:commandButton value="Guardar"
                                 action="#{userController.saveUser()}"
                                 update="@form"
                                 process="@form"
                                 icon="pi pi-check"
                                 styleClass="p-button-success"/>
                
                <p:commandButton value="Cancelar"
                                 action="#{userController.cancel()}"
                                 immediate="true"
                                 icon="pi pi-times"
                                 styleClass="p-button-secondary"/>
                
            </p:panel>
            
        </h:form>
    </ui:define>
</ui:composition>
```
# Controller for Form
```
@Named
@ViewScoped
public class UserController implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private UserDTO user;
    private List<SelectItem> roleOptions;
    
    @Inject
    private UserService userService;
    
    @Inject
    private RoleService roleService;
    
    @PostConstruct
    public void init() {
        user = new UserDTO();
        loadRoleOptions();
    }
    
    private void loadRoleOptions() {
        roleOptions = roleService.findAllActiveRoles().stream()
            .map(role -> new SelectItem(role.getId(), role.getName()))
            .collect(Collectors.toList());
    }
    
    public void saveUser() {
        try {
            userService.saveUser(user);
            FacesUtil.addSuccessMessage("Usuario guardado exitosamente");
            resetForm();
        } catch (BusinessException e) {
            FacesUtil.addErrorMessage("Error al guardar: " + e.getMessage());
        } catch (Exception e) {
            FacesUtil.addErrorMessage("Error inesperado");
            logger.error("Save user error", e);
        }
    }
    
    private void resetForm() {
        user = new UserDTO();
        // Mantiene las opciones cargadas
    }
    
    public void cancel() {
        resetForm();
        FacesUtil.addInfoMessage("Operación cancelada");
    }
    
    // Getters y Setters
    public UserDTO getUser() { return user; }
    public void setUser(UserDTO user) { this.user = user; }
    public List<SelectItem> getRoleOptions() { return roleOptions; }
}
```
# UI Component Best Practices
DataTable Example

```
<p:dataTable id="userTable"
             value="#{userController.userList}"
             var="user"
             paginator="true"
             rows="10"
             paginatorTemplate="{CurrentPageReport} {FirstPageLink} {PreviousPageLink} {PageLinks} {NextPageLink} {LastPageLink} {RowsPerPageDropdown}"
             rowsPerPageTemplate="5,10,15"
             emptyMessage="No hay usuarios"
             filteredValue="#{userController.filteredUsers}">
    
    <p:column headerText="ID" sortBy="#{user.id}" filterBy="#{user.id}">
        <h:outputText value="#{user.id}"/>
    </p:column>
    
    <p:column headerText="Nombre" sortBy="#{user.fullName}" filterBy="#{user.fullName}">
        <h:outputText value="#{user.fullName}"/>
    </p:column>
    
    <p:column headerText="Acciones" style="width:120px">
        <p:commandButton icon="pi pi-pencil"
                         title="Editar"
                         action="#{userController.editUser(user)}"
                         update=":userForm"
                         oncomplete="PF('userDialog').show()"/>
        
        <p:commandButton icon="pi pi-trash"
                         title="Eliminar"
                         action="#{userController.deleteUser(user)}"
                         update="@form"
                         onclick="return confirm('¿Eliminar usuario?')"/>
    </p:column>
</p:dataTable>
```
# Utility Classes & Helpers

```
// ✅ FacesUtil.java - Helper para JSF
public class FacesUtil {
    
    private FacesUtil() {
        // Utility class - no instantiation
    }
    
    public static void addSuccessMessage(String message) {
        addMessage(FacesMessage.SEVERITY_INFO, "Éxito", message);
    }
    
    public static void addErrorMessage(String message) {
        addMessage(FacesMessage.SEVERITY_ERROR, "Error", message);
    }
    
    public static void addInfoMessage(String message) {
        addMessage(FacesMessage.SEVERITY_INFO, "Información", message);
    }
    
    private static void addMessage(FacesMessage.Severity severity, 
                                   String summary, String detail) {
        FacesContext.getCurrentInstance()
            .addMessage(null, new FacesMessage(severity, summary, detail));
    }
    
    public static String getRequestParameter(String key) {
        return FacesContext.getCurrentInstance()
            .getExternalContext()
            .getRequestParameterMap()
            .get(key);
    }
}
```

# Key Recommendations

```
// ✅ Siempre validar en backend
public void saveUser(@Valid UserDTO user) {
    // Validación adicional del negocio
    if (userService.emailExists(user.getEmail())) {
        throw new BusinessException("Email ya registrado");
    }
}

// ✅ Prevenir XSS
<h:outputText value="#{userController.user.bio}" escape="true"/>
```

# Performance

```
// ✅ Lazy loading para tablas grandes
private LazyDataModel<UserDTO> lazyModel;

@PostConstruct
public void init() {
    lazyModel = new LazyUserDataModel(userService);
}

// ✅ Cachear datos estáticos
@Named
@ApplicationScoped
public class StaticDataCache {
    private List<Country> countries;
    
    @PostConstruct
    public void loadCountries() {
        countries = countryRepository.findAll();
    }
}
```
# Error Handling
```
// ✅ Global exception handler
public void handleException() {
    FacesContext context = FacesContext.getCurrentInstance();
    Exception exception = (Exception) context.getExternalContext()
        .getRequestMap().get("javax.servlet.error.exception");
    
    // Log y mensaje amigable
    logger.error("Error en aplicación", exception);
    FacesUtil.addErrorMessage("Ocurrió un error. Contacte soporte.");
}
```

#  JoinFaces Configuration

```
# application.yml
joinfaces:
  jsf:
    project-stage: Production
    state-saving-method: server
    
  primefaces:
    theme: nova-light
    font-awesome: true
    uploader: native
    
  security:
    enable-csrf-token: true
    
  spring-boot:
    admin:
      enabled: false
```
# TEMPLATES Y COMPONENTES REUSABLES
```
<!-- template.xhtml -->
<!DOCTYPE html>
<html lang="es">
<h:head>
    <title>#{appName} - #{view.viewId}</title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
</h:head>

<h:body>
    <header role="banner">
        <!-- Header content -->
    </header>
    
    <main role="main">
        <ui:insert name="content">
            <!-- Contenido dinámico -->
        </ui:insert>
    </main>
    
    <footer role="contentinfo">
        <!-- Footer content -->
    </footer>
</h:body>
</html>
```


# Template CRUD Completo con Mejores Prácticas
1. Template Principal (template.xhtml)


```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
      xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>#{msgs['app.name']} - #{view.viewId}</title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    
    <!-- Favicon -->
    <link rel="icon" type="image/x-icon" href="#{resource['images/favicon.ico']}" />
    
    <!-- CSS Resources -->
    <h:outputStylesheet library="css" name="app.css" />
    
    <!-- JS Resources -->
    <h:outputScript library="js" name="app.js" />
    
    <!-- Dynamic Title for Pages -->
    <f:metadata>
        <f:viewAction action="#{navigationController.updatePageTitle}" />
    </f:metadata>
</h:head>

<h:body>
    <!-- Loading Indicator -->
    <p:ajaxStatus onstart="PF('loadingDialog').show()" 
                  onsuccess="PF('loadingDialog').hide()"/>
    
    <!-- Global Loading Dialog -->
    <p:dialog modal="true" widgetVar="loadingDialog" header="Cargando" 
              draggable="false" closable="false" resizable="false">
        <p:graphicImage library="images" name="loading.gif" />
    </p:dialog>

    <!-- Global Messages (Top of Page) -->
    <div class="global-messages">
        <p:growl id="globalGrowl" showDetail="true" life="5000" sticky="false" />
        <p:messages id="globalMessages" showDetail="true" closable="true" 
                    globalOnly="true" redisplay="false" />
    </div>

    <!-- Main Layout -->
    <div class="layout-wrapper">
        <!-- Header -->
        <header class="layout-header">
            <div class="header-content">
                <h:graphicImage library="images" name="logo.png" 
                               styleClass="app-logo" />
                <h1>#{msgs['app.name']}</h1>
                
                <!-- User Session Info -->
                <div class="user-info" rendered="#{userSessionController.loggedIn}">
                    <p:commandLink styleClass="user-profile">
                        <i class="pi pi-user"></i>
                        #{userSessionController.currentUser.fullName}
                    </p:commandLink>
                    <p:commandButton value="Salir" 
                                     action="#{authController.logout}"
                                     icon="pi pi-sign-out"
                                     styleClass="logout-btn" />
                </div>
            </div>
        </header>

        <!-- Navigation Menu -->
        <nav class="layout-sidebar" rendered="#{userSessionController.loggedIn}">
            <p:menu model="#{navigationController.menuModel}" />
        </nav>

        <!-- Main Content -->
        <main class="layout-content">
            <!-- Page Title -->
            <div class="page-header">
                <h2>#{navigationController.currentPageTitle}</h2>
                <p:breadCrumb model="#{navigationController.breadcrumbModel}" />
            </div>

            <!-- Dynamic Content -->
            <ui:insert name="content">
                <!-- Content will be inserted here from each page -->
            </ui:insert>
        </main>

        <!-- Footer -->
        <footer class="layout-footer">
            <p>#{msgs['app.copyright']} #{currentYear}</p>
            <p>Versión: #{applicationController.appVersion}</p>
        </footer>
    </div>

    <!-- JavaScript Initialization -->
    <h:outputScript>
        $(document).ready(function() {
            // Initialize tooltips
            $('[title]').tooltip();
            
            // Handle session timeout
            var sessionTimeout = #{session.maxInactiveInterval} * 1000;
            setTimeout(function() {
                showSessionWarning();
            }, sessionTimeout - 60000); // Warn 1 minute before timeout
        });
    </h:outputScript>
</h:body>
</html>
```


2. CRUD Template Page (crud-template.xhtml)

```
<ui:composition template="/WEB-INF/templates/template.xhtml"
                xmlns="http://www.w3.org/1999/xhtml"
                xmlns:h="http://xmlns.jcp.org/jsf/html"
                xmlns:p="http://primefaces.org/ui"
                xmlns:f="http://xmlns.jcp.org/jsf/core"
                xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
                xmlns:c="http://xmlns.jcp.org/jsf/composite">

    <!-- Define metadata for this CRUD -->
    <f:metadata>
        <f:viewAction action="#{crudController.init}"
                      onPostback="false" />
        <f:viewParam name="id" value="#{crudController.entityId}" />
        <f:event type="preRenderView" 
                 listener="#{crudController.checkPermissions}" />
    </f:metadata>

    <ui:define name="content">
        <!-- CRUD Container -->
        <div class="crud-container">
            
            <!-- Action Toolbar -->
            <div class="action-toolbar">
                <p:toolbar>
                    <p:toolbarGroup>
                        <!-- Create Button -->
                        <p:commandButton value="#{msgs['crud.create']}"
                                         icon="pi pi-plus"
                                         action="#{crudController.prepareCreate}"
                                         oncomplete="PF('entityDialog').show()"
                                         rendered="#{crudController.canCreate}"
                                         styleClass="p-button-success" />
                        
                        <!-- Export Buttons -->
                        <p:commandButton icon="pi pi-file-excel"
                                         title="Exportar a Excel"
                                         onclick="PF('dataTable').exportAsXLS()"
                                         rendered="#{not empty crudController.entityList}" />
                        
                        <p:commandButton icon="pi pi-file-pdf"
                                         title="Exportar a PDF"
                                         onclick="PF('dataTable').exportAsPDF()"
                                         rendered="#{not empty crudController.entityList}" />
                    </p:toolbarGroup>
                    
                    <p:toolbarGroup align="right">
                        <!-- Search -->
                        <p:inputText id="globalFilter"
                                     placeholder="Buscar..."
                                     value="#{crudController.globalFilter}"
                                     style="width: 250px">
                            <p:ajax event="keyup" 
                                    listener="#{crudController.onSearch}"
                                    update="dataTable"
                                    delay="300" />
                        </p:inputText>
                        
                        <!-- Refresh -->
                        <p:commandButton icon="pi pi-refresh"
                                         title="Refrescar"
                                         action="#{crudController.refresh}"
                                         update="dataTable" />
                    </p:toolbarGroup>
                </p:toolbar>
            </div>
            
            <!-- Data Table -->
            <div class="data-table-container">
                <p:dataTable id="dataTable"
                             widgetVar="dataTable"
                             value="#{crudController.lazyModel}"
                             var="entity"
                             paginator="true"
                             rows="10"
                             rowsPerPageTemplate="5,10,15,20"
                             paginatorTemplate="{CurrentPageReport} {FirstPageLink} {PreviousPageLink} {PageLinks} {NextPageLink} {LastPageLink} {RowsPerPageDropdown}"
                             emptyMessage="#{msgs['crud.no.records']}"
                             filteredValue="#{crudController.filteredList}"
                             lazy="true">
                    
                    <!-- Dynamic Columns (Example) -->
                    <p:column headerText="ID" 
                              sortBy="#{entity.id}" 
                              filterBy="#{entity.id}"
                              style="width: 80px">
                        <h:outputText value="#{entity.id}" />
                    </p:column>
                    
                    <p:column headerText="Nombre" 
                              sortBy="#{entity.name}" 
                              filterBy="#{entity.name}">
                        <h:outputText value="#{entity.name}" />
                    </p:column>
                    
                    <p:column headerText="Estado" 
                              sortBy="#{entity.active}" 
                              style="width: 100px">
                        <p:selectBooleanCheckbox value="#{entity.active}"
                                                 disabled="true" />
                    </p:column>
                    
                    <!-- Actions Column -->
                    <p:column headerText="Acciones" 
                              style="width: 150px; text-align: center">
                        <p:commandButton icon="pi pi-eye"
                                         title="Ver detalles"
                                         action="#{crudController.viewDetails(entity)}"
                                         oncomplete="PF('detailDialog').show()"
                                         styleClass="p-button-info p-button-outlined" />
                        
                        <p:commandButton icon="pi pi-pencil"
                                         title="Editar"
                                         action="#{crudController.prepareEdit(entity)}"
                                         oncomplete="PF('entityDialog').show()"
                                         rendered="#{crudController.canEdit}"
                                         styleClass="p-button-warning p-button-outlined" />
                        
                        <p:commandButton icon="pi pi-trash"
                                         title="Eliminar"
                                         action="#{crudController.prepareDelete(entity)}"
                                         oncomplete="PF('confirmDialog').show()"
                                         rendered="#{crudController.canDelete}"
                                         styleClass="p-button-danger p-button-outlined" />
                    </p:column>
                    
                    <!-- Row Expansion for Details -->
                    <p:rowExpansion>
                        <div class="row-details">
                            <p:fieldset legend="Detalles">
                                <!-- Dynamic details based on entity -->
                                <p:outputLabel value="Creado por: " />
                                <p:outputText value="#{entity.createdBy}" />
                                <br />
                                <p:outputLabel value="Fecha creación: " />
                                <p:outputText value="#{entity.createdDate}">
                                    <f:convertDateTime pattern="dd/MM/yyyy HH:mm" />
                                </p:outputText>
                            </p:fieldset>
                        </div>
                    </p:rowExpansion>
                </p:dataTable>
            </div>
        </div>
        
        <!-- MODALS SECTION -->
        
        <!-- Create/Edit Modal -->
        <ui:include src="/WEB-INF/includes/modals/entity-form-modal.xhtml" />
        
        <!-- Delete Confirmation Modal -->
        <ui:include src="/WEB-INF/includes/modals/confirm-dialog.xhtml" />
        
        <!-- Detail View Modal -->
        <ui:include src="/WEB-INF/includes/modals/detail-view-modal.xhtml" />
        
    </ui:define>
</ui:composition>
```
# Componentes Modales Separados
3. Modal de Formulario (entity-form-modal.xhtml)

```
<ui:composition xmlns="http://www.w3.org/1999/xhtml"
                xmlns:h="http://xmlns.jcp.org/jsf/html"
                xmlns:p="http://primefaces.org/ui"
                xmlns:f="http://xmlns.jcp.org/jsf/core">

    <!-- Create/Edit Modal -->
    <p:dialog header="#{crudController.entity.id == null ? 'Crear' : 'Editar'} Registro"
              widgetVar="entityDialog"
              modal="true"
              appendTo="@(body)"
              resizable="false"
              draggable="false"
              closeOnEscape="true"
              styleClass="form-dialog">
        
        <h:form id="entityForm">
            <!-- Validation Messages -->
            <p:messages id="formMessages" showDetail="true" closable="true" />
            
            <!-- Form Content -->
            <div class="form-container">
                <p:panelGrid columns="2" layout="grid" styleClass="form-grid">
                    
                    <!-- Required Field with Validation -->
                    <p:outputLabel for="name" value="Nombre *" />
                    <p:inputText id="name" 
                                 value="#{crudController.entity.name}"
                                 required="true"
                                 requiredMessage="El nombre es requerido"
                                 maxlength="100">
                        <f:validateLength minimum="3" maximum="100" />
                        <p:ajax event="blur" update="nameMessage" />
                    </p:inputText>
                    <p:message for="name" id="nameMessage" />
                    
                    <!-- Email Field -->
                    <p:outputLabel for="email" value="Email *" />
                    <p:inputText id="email"
                                 value="#{crudController.entity.email}"
                                 required="true"
                                 requiredMessage="Email es requerido">
                        <f:validateRegex pattern="^[A-Za-z0-9+_.-]+@(.+)$" />
                        <p:ajax event="blur" 
                                listener="#{crudController.validateEmail}"
                                update="emailMessage" />
                    </p:inputText>
                    <p:message for="email" id="emailMessage" />
                    
                    <!-- Select One Menu -->
                    <p:outputLabel for="type" value="Tipo *" />
                    <p:selectOneMenu id="type"
                                     value="#{crudController.entity.type}"
                                     required="true"
                                     converter="#{typeConverter}">
                        <f:selectItem itemLabel="Seleccione..." noSelectionOption="true" />
                        <f:selectItems value="#{crudController.typeOptions}"
                                       var="type"
                                       itemLabel="#{type.label}"
                                       itemValue="#{type}" />
                        <p:ajax event="change" update="@form" />
                    </p:selectOneMenu>
                    
                    <!-- Calendar -->
                    <p:outputLabel for="birthDate" value="Fecha Nacimiento" />
                    <p:calendar id="birthDate"
                                value="#{crudController.entity.birthDate}"
                                pattern="dd/MM/yyyy"
                                locale="es"
                                yearRange="c-100:c"
                                navigator="true" />
                    
                    <!-- TextArea -->
                    <p:outputLabel for="description" value="Descripción" />
                    <p:textarea id="description"
                                value="#{crudController.entity.description}"
                                rows="3"
                                cols="40"
                                maxlength="500"
                                counter="display"
                                counterTemplate="{0} caracteres restantes" />
                    
                    <!-- Toggle Switch -->
                    <p:outputLabel for="active" value="Activo" />
                    <p:selectBooleanCheckbox id="active"
                                             value="#{crudController.entity.active}">
                        <p:ajax event="change" update="@form" />
                    </p:selectBooleanCheckbox>
                </p:panelGrid>
            </div>
            
            <!-- Dialog Footer -->
            <f:facet name="footer">
                <p:commandButton value="Guardar"
                                 action="#{crudController.save}"
                                 update=":form:dataTable :form:globalGrowl"
                                 process="@form"
                                 oncomplete="handleSaveResponse(xhr, status, args)"
                                 styleClass="p-button-success" />
                
                <p:commandButton value="Cancelar"
                                 onclick="PF('entityDialog').hide()"
                                 immediate="true"
                                 styleClass="p-button-secondary" />
            </f:facet>
        </h:form>
        
        <!-- JavaScript for Dialog -->
        <h:outputScript>
            function handleSaveResponse(xhr, status, args) {
                if (args.validationFailed || args.error) {
                    // Show error messages
                    PF('entityDialog').jq.effect('shake', { times: 3 }, 500);
                } else {
                    // Success - close dialog and show success message
                    PF('entityDialog').hide();
                    PF('globalGrowl').show([{
                        severity: 'info',
                        summary: 'Éxito',
                        detail: args.saveMessage || 'Registro guardado correctamente'
                    }]);
                }
            }
        </h:outputScript>
    </p:dialog>
</ui:composition>
```

 4. Modal de Confirmación (confirm-dialog.xhtml)

```
<ui:composition xmlns="http://www.w3.org/1999/xhtml"
                xmlns:p="http://primefaces.org/ui"
                xmlns:h="http://xmlns.jcp.org/jsf/html">

    <!-- Delete Confirmation Modal -->
    <p:confirmDialog header="Confirmar Eliminación"
                     widgetVar="confirmDialog"
                     message="¿Está seguro de eliminar este registro?"
                     severity="alert"
                     appendTo="@(body)">
        
        <p:commandButton value="Sí"
                         action="#{crudController.delete}"
                         update=":form:dataTable :form:globalGrowl"
                         oncomplete="PF('confirmDialog').hide()"
                         styleClass="p-button-danger" />
        
        <p:commandButton value="No"
                         onclick="PF('confirmDialog').hide()"
                         styleClass="p-button-secondary" />
    </p:confirmDialog>
</ui:composition>
```
# Controller CRUD Base Abstracto
5. Abstract CRUD Controller (BaseCrudController.java)

```
package com.empresa.proyecto.controller;

import com.empresa.proyecto.dto.BaseDTO;
import com.empresa.proyecto.service.BaseService;
import com.empresa.proyecto.util.FacesUtil;
import lombok.Getter;
import lombok.Setter;
import org.primefaces.model.LazyDataModel;
import org.primefaces.model.SortOrder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.annotation.PostConstruct;
import javax.faces.view.ViewScoped;
import javax.inject.Inject;
import javax.inject.Named;
import java.io.Serializable;
import java.util.List;
import java.util.Map;

/**
 * Controlador base abstracto para operaciones CRUD
 * @param <T> Tipo del DTO
 * @param <S> Tipo del servicio
 */
@Named
@ViewScoped
public abstract class BaseCrudController<T extends BaseDTO, S extends BaseService<T>> 
        implements Serializable {
    
    private static final long serialVersionUID = 1L;
    protected final Logger logger = LoggerFactory.getLogger(getClass());
    
    @Inject
    protected S service;
    
    @Getter @Setter
    protected T entity;
    
    @Getter @Setter
    protected T selectedEntity;
    
    @Getter
    protected List<T> entityList;
    
    @Getter
    protected List<T> filteredList;
    
    @Getter @Setter
    protected LazyDataModel<T> lazyModel;
    
    @Getter @Setter
    protected String globalFilter;
    
    @Getter @Setter
    protected Long entityId;
    
    @Getter
    protected boolean editMode = false;
    
    @Getter
    protected boolean canCreate = true;
    
    @Getter
    protected boolean canEdit = true;
    
    @Getter
    protected boolean canDelete = true;
    
    @PostConstruct
    public void init() {
        initEntity();
        initLazyModel();
        loadPermissions();
    }
    
    /**
     * Inicializar la entidad
     */
    protected void initEntity() {
        entity = createNewInstance();
    }
    
    /**
     * Crear nueva instancia del DTO
     */
    protected abstract T createNewInstance();
    
    /**
     * Inicializar el modelo lazy
     */
    protected void initLazyModel() {
        lazyModel = new LazyDataModel<T>() {
            @Override
            public List<T> load(int first, int pageSize, 
                              String sortField, SortOrder sortOrder,
                              Map<String, Object> filters) {
                
                try {
                    // Configurar paginación
                    setRowCount(service.count(filters));
                    
                    // Obtener datos paginados
                    List<T> results = service.findPaginated(
                        first, pageSize, sortField, sortOrder, filters);
                    
                    return results;
                } catch (Exception e) {
                    logger.error("Error loading lazy data", e);
                    FacesUtil.addErrorMessage("Error al cargar datos");
                    return List.of();
                }
            }
        };
    }
    
    /**
     * Preparar creación de nuevo registro
     */
    public void prepareCreate() {
        entity = createNewInstance();
        editMode = false;
        logger.info("Preparing to create new entity");
    }
    
    /**
     * Preparar edición de registro
     */
    public void prepareEdit(T entityToEdit) {
        this.entity = service.findById(entityToEdit.getId());
        this.selectedEntity = entityToEdit;
        editMode = true;
        logger.info("Preparing to edit entity ID: {}", entity.getId());
    }
    
    /**
     * Guardar registro (create or update)
     */
    public void save() {
        try {
            validateBeforeSave();
            
            if (editMode) {
                entity = service.update(entity);
                FacesUtil.addSuccessMessage("Registro actualizado exitosamente");
                logger.info("Entity updated: {}", entity.getId());
            } else {
                entity = service.create(entity);
                FacesUtil.addSuccessMessage("Registro creado exitosamente");
                logger.info("Entity created: {}", entity.getId());
            }
            
            // Resetear después de guardar
            resetAfterSave();
            
        } catch (BusinessException e) {
            FacesUtil.addErrorMessage(e.getMessage());
            logger.warn("Business exception on save: {}", e.getMessage());
        } catch (Exception e) {
            FacesUtil.addErrorMessage("Error al guardar el registro");
            logger.error("Error saving entity", e);
            throw e;
        }
    }
    
    /**
     * Validar antes de guardar
     */
    protected void validateBeforeSave() {
        // Validaciones específicas se implementan en clases hijas
        if (entity.getName() == null || entity.getName().trim().isEmpty()) {
            throw new BusinessException("El nombre es requerido");
        }
    }
    
    /**
     * Resetear después de guardar
     */
    protected void resetAfterSave() {
        entity = createNewInstance();
        editMode = false;
        selectedEntity = null;
        // Actualizar lista
        refresh();
    }
    
    /**
     * Preparar eliminación
     */
    public void prepareDelete(T entityToDelete) {
        this.selectedEntity = entityToDelete;
    }
    
    /**
     * Eliminar registro
     */
    public void delete() {
        try {
            if (selectedEntity != null && selectedEntity.getId() != null) {
                service.delete(selectedEntity.getId());
                FacesUtil.addSuccessMessage("Registro eliminado exitosamente");
                logger.info("Entity deleted: {}", selectedEntity.getId());
                selectedEntity = null;
                refresh();
            }
        } catch (BusinessException e) {
            FacesUtil.addErrorMessage(e.getMessage());
            logger.warn("Business exception on delete: {}", e.getMessage());
        } catch (Exception e) {
            FacesUtil.addErrorMessage("Error al eliminar el registro");
            logger.error("Error deleting entity", e);
        }
    }
    
    /**
     * Refrescar datos
     */
    public void refresh() {
        entityList = service.findAll();
        logger.debug("Data refreshed, {} records loaded", entityList.size());
    }
    
    /**
     * Buscar con filtro global
     */
    public void onSearch() {
        if (globalFilter != null && !globalFilter.trim().isEmpty()) {
            filteredList = service.search(globalFilter);
        } else {
            filteredList = null;
        }
    }
    
    /**
     * Cargar permisos del usuario
     */
    protected void loadPermissions() {
        // Implementar lógica de permisos según roles
        // Ejemplo:
        // canCreate = userSessionController.hasPermission("CREATE");
        // canEdit = userSessionController.hasPermission("EDIT");
        // canDelete = userSessionController.hasPermission("DELETE");
    }
    
    /**
     * Ver detalles del registro
     */
    public void viewDetails(T entity) {
        this.selectedEntity = entity;
        logger.debug("Viewing details for entity ID: {}", entity.getId());
    }
}
```

6. Ejemplo de Controller Concreto (UsuarioController.java)
```
package com.empresa.proyecto.controller;

import com.empresa.proyecto.dto.UsuarioDTO;
import com.empresa.proyecto.service.UsuarioService;
import lombok.Getter;
import org.primefaces.model.FilterMeta;
import org.springframework.beans.factory.annotation.Autowired;

import javax.annotation.PostConstruct;
import javax.faces.view.ViewScoped;
import javax.inject.Named;
import java.util.List;
import java.util.Map;

@Named
@ViewScoped
public class UsuarioController extends BaseCrudController<UsuarioDTO, UsuarioService> {
    
    @Getter
    private List<SelectItem> roleOptions;
    
    @Getter
    private List<SelectItem> departmentOptions;
    
    @Autowired
    private RoleService roleService;
    
    @Autowired
    private DepartmentService departmentService;
    
    @PostConstruct
    @Override
    public void init() {
        super.init();
        loadOptions();
    }
    
    @Override
    protected UsuarioDTO createNewInstance() {
        return new UsuarioDTO();
    }
    
    private void loadOptions() {
        // Cargar opciones para selects
        roleOptions = roleService.findAllActive().stream()
            .map(role -> new SelectItem(role.getId(), role.getNombre()))
            .collect(Collectors.toList());
            
        departmentOptions = departmentService.findAll().stream()
            .map(dept -> new SelectItem(dept.getId(), dept.getNombre()))
            .collect(Collectors.toList());
    }
    
    @Override
    protected void validateBeforeSave() {
        super.validateBeforeSave();
        
        // Validaciones específicas de Usuario
        if (entity.getEmail() == null || !entity.getEmail().contains("@")) {
            throw new BusinessException("Email inválido");
        }
        
        if (entity.getPassword() != null && entity.getPassword().length() < 6) {
            throw new BusinessException("La contraseña debe tener al menos 6 caracteres");
        }
    }
    
    public void validateEmail() {
        if (entity.getEmail() != null && !editMode) {
            boolean exists = service.emailExists(entity.getEmail());
            if (exists) {
                FacesUtil.addErrorMessage("El email ya está registrado");
            }
        }
    }
    
    // Métodos específicos de Usuario
    public void resetPassword() {
        try {
            service.resetPassword(selectedEntity.getId());
            FacesUtil.addSuccessMessage("Contraseña reiniciada exitosamente");
        } catch (Exception e) {
            FacesUtil.addErrorMessage("Error al reiniciar contraseña");
            logger.error("Error resetting password", e);
        }
    }
    
    public void toggleStatus() {
        try {
            service.toggleStatus(selectedEntity.getId());
            String newStatus = selectedEntity.isActive() ? "desactivado" : "activado";
            FacesUtil.addSuccessMessage("Usuario " + newStatus + " exitosamente");
            refresh();
        } catch (Exception e) {
            FacesUtil.addErrorMessage("Error al cambiar estado");
            logger.error("Error toggling status", e);
        }
    }
}
```

# Manejo de Mensajes - Mejores Prácticas
7. Clase de Utilidad para Mensajes (MessageHandler.java)
```
package com.empresa.proyecto.util;

import org.primefaces.PrimeFaces;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.faces.application.FacesMessage;
import javax.faces.context.FacesContext;
import java.util.ResourceBundle;

/**
 * Utilidad para manejo consistente de mensajes
 */
public class MessageHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(MessageHandler.class);
    private static final String BUNDLE_NAME = "messages";
    
    private MessageHandler() {
        // Utility class
    }
    
    /**
     * Obtener mensaje del bundle
     */
    public static String getMessage(String key) {
        try {
            FacesContext context = FacesContext.getCurrentInstance();
            ResourceBundle bundle = context.getApplication()
                .getResourceBundle(context, "msgs");
            return bundle.getString(key);
        } catch (Exception e) {
            logger.warn("Message key not found: {}", key);
            return key;
        }
    }
    
    /**
     * Mensaje de éxito
     */
    public static void addSuccess(String detail) {
        addMessage(FacesMessage.SEVERITY_INFO, 
                  getMessage("message.success.title"), 
                  detail);
    }
    
    /**
     * Mensaje de error
     */
    public static void addError(String detail) {
        addMessage(FacesMessage.SEVERITY_ERROR,
                  getMessage("message.error.title"),
                  detail);
    }
    
    /**
     * Mensaje de advertencia
     */
    public static void addWarn(String detail) {
        addMessage(FacesMessage.SEVERITY_WARN,
                  getMessage("message.warn.title"),
                  detail);
    }
    
    /**
     * Mensaje de información
     */
    public static void addInfo(String detail) {
        addMessage(FacesMessage.SEVERITY_INFO,
                  getMessage("message.info.title"),
                  detail);
    }
    
    /**
     * Mensaje con parámetros
     */
    public static void addSuccess(String key, Object... params) {
        String message = String.format(getMessage(key), params);
        addSuccess(message);
    }
    
    /**
     * Mostrar mensaje en growl
     */
    public static void showGrowl(FacesMessage.Severity severity, 
                                 String summary, String detail) {
        FacesContext.getCurrentInstance().addMessage(null, 
            new FacesMessage(severity, summary, detail));
        
        // También ejecutar script para mostrar growl
        PrimeFaces.current().executeScript(
            "PF('globalGrowl').show([{" +
            "severity: '" + getSeverityString(severity) + "'," +
            "summary: '" + summary + "'," +
            "detail: '" + detail + "'" +
            "}]);"
        );
    }
    
    /**
     * Manejo de excepciones
     */
    public static void handleException(Exception e, String userMessage) {
        logger.error("Error en aplicación: {}", e.getMessage(), e);
        
        // Mensaje amigable para usuario
        addError(userMessage != null ? userMessage : 
                getMessage("error.generic"));
        
        // En desarrollo, mostrar más detalles
        if (isDevelopment()) {
            addInfo("Detalle técnico: " + e.getMessage());
        }
    }
    
    /**
     * Validación específica
     */
    public static boolean validateRequired(Object value, String fieldName) {
        if (value == null || 
            (value instanceof String && ((String) value).trim().isEmpty())) {
            addError(String.format(getMessage("validation.required"), fieldName));
            return false;
        }
        return true;
    }
    
    private static void addMessage(FacesMessage.Severity severity, 
                                   String summary, String detail) {
        FacesContext.getCurrentInstance()
            .addMessage(null, new FacesMessage(severity, summary, detail));
    }
    
    private static String getSeverityString(FacesMessage.Severity severity) {
        if (severity == FacesMessage.SEVERITY_INFO) return "info";
        if (severity == FacesMessage.SEVERITY_WARN) return "warn";
        if (severity == FacesMessage.SEVERITY_ERROR) return "error";
        if (severity == FacesMessage.SEVERITY_FATAL) return "fatal";
        return "info";
    }
    
    private static boolean isDevelopment() {
        return FacesContext.getCurrentInstance()
            .getApplication().getProjectStage() == 
            ProjectStage.Development;
    }
}

```

# Cuando usar p:commandButton vs Otros

```
// ✅ p:commandButton - CUANDO USAR:
// 1. Acciones que requieren procesamiento en servidor
// 2. Envío de formularios
// 3. Operaciones CRUD
// 4. Navegación con procesamiento
<p:commandButton value="Guardar" 
                 action="#{controller.save()}" 
                 update="@form" 
                 process="@form" />

// ✅ h:commandButton - CUANDO USAR:
// 1. Formularios simples sin AJAX
// 2. Cuando necesitas full page refresh intencional
// 3. Compatibilidad con navegadores antiguos
<h:commandButton value="Enviar" action="#{controller.submit()}" />

// ✅ p:button - CUANDO USAR:
// 1. Navegación simple (GET requests)
// 2. Links que parecen botones
// 3. No requiere procesamiento de formulario
<p:button value="Ver Reporte" outcome="/reports/daily" />

// ✅ h:button - CUANDO USAR:
// 1. Navegación básica
// 2. Similar a p:button pero sin estilos PrimeFaces
<h:button value="Inicio" outcome="/index" />

// ✅ p:commandLink - CUANDO USAR:
// 1. Links que ejecutan acción
// 2. Menús con acciones
// 3. Cuando quieres estilo de link con funcionalidad de botón
<p:commandLink action="#{controller.delete()}" 
               update="table">
    <h:outputText value="Eliminar" />
</p:commandLink>

// ❌ EVITAR mezclar en el mismo formulario:
// No mezclar p:commandButton con h:commandButton
// Usar consistencia en toda la aplicación
```
# Ejemplos Prácticos:

```
<!-- ✅ CORRECTO: p:commandButton para CRUD -->
<p:commandButton value="Guardar" 
                 action="#{usuarioController.save}"
                 update="@form :messages"
                 process="@form"
                 icon="pi pi-save"
                 styleClass="p-button-success" />

<!-- ✅ CORRECTO: p:button para navegación -->
<p:button value="Volver al Listado"
          outcome="/usuarios/list"
          icon="pi pi-arrow-left" />

<!-- ✅ CORRECTO: p:commandLink para acciones en tabla -->
<p:commandLink action="#{usuarioController.edit(usuario)}"
               update=":editForm"
               oncomplete="PF('editDialog').show()">
    <i class="pi pi-pencil" title="Editar" />
</p:commandLink>

<!-- ✅ CORRECTO: Botón con confirmación -->
<p:commandButton value="Eliminar"
                 action="#{usuarioController.delete}"
                 update="@form"
                 onclick="return confirm('¿Está seguro?')"
                 icon="pi pi-trash"
                 styleClass="p-button-danger" />
```

# Estructura Final de Archivos
```
src/main/webapp/
├── WEB-INF/
│   ├── web.xml
│   ├── faces-config.xml
│   ├── templates/
│   │   ├── template.xhtml          # Template principal
│   │   └── crud-template.xhtml     # Template CRUD
│   ├── includes/
│   │   └── modals/
│   │       ├── entity-form-modal.xhtml
│   │       ├── confirm-dialog.xhtml
│   │       └── detail-view-modal.xhtml
│   └── resources/
│       └── empresa/
│           ├── css/
│           │   ├── app.css
│           │   └── components.css
│           ├── js/
│           │   └── app.js
│           ├── images/
│           └── layout/
│               └── template.xhtml
├── usuarios/
│   ├── list.xhtml
│   ├── form.xhtml
│   └── detail.xhtml
├── productos/
│   ├── list.xhtml
│   ├── form.xhtml
│   └── detail.xhtml
└── index.xhtml

```

# Implementación de Loading Indicators en PrimeFaces

1. Loading Global (Para toda la aplicación)

```
<!-- En template.xhtml -->
<p:ajaxStatus onstart="PF('loadingDialog').show()" 
              onsuccess="PF('loadingDialog').hide()"
              onerror="PF('loadingDialog').hide()"/>

<!-- Dialog de loading global -->
<p:dialog modal="true" widgetVar="loadingDialog" 
          draggable="false" closable="false" resizable="false"
          showHeader="false" style="background: transparent; border: none;">
    <div class="loading-container">
        <!-- Spinner de PrimeFaces -->
        <p:graphicImage library="primefaces" name="loading.svg" 
                       style="width: 64px; height: 64px;"/>
        <p style="margin-top: 10px; color: #2196F3;">Procesando...</p>
    </div>
</p:dialog>

<!-- O usando PrimeFaces Default Spinner -->
<p:blockUI widgetVar="globalBlockUI" block="body" trigger="@(body)">
    <div class="blockui-content">
        <p:graphicImage library="primefaces" name="loading.svg" />
        <h4>Procesando solicitud...</h4>
    </div>
</p:blockUI>
```

2. Loading por Botón/Acción Específica
```
<!-- Botón con loading específico -->
<p:commandButton value="Procesar Datos" 
                 action="#{reportController.generateReport}"
                 icon="pi pi-spinner"
                 styleClass="btn-loading"
                 onstart="PF('reportLoading').show()"
                 oncomplete="PF('reportLoading').hide()"
                 update="reportPanel"/>

<!-- Loading específico para este botón -->
<p:blockUI widgetVar="reportLoading" block="reportPanel">
    <div class="custom-loading">
        <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">Cargando...</span>
        </div>
        <p>Generando reporte...</p>
    </div>
</p:blockUI>
```

4. Loading con ProgressBar (Para operaciones largas)


```
<!-- En el controller -->
@Named
@ViewScoped
public class ImportController implements Serializable {
    private Integer progress;
    private boolean importing;
    
    public void startImport() {
        importing = true;
        progress = 0;
        
        // Simular proceso largo
        new Thread(() -> {
            try {
                for (int i = 0; i <= 100; i++) {
                    Thread.sleep(100);
                    progress = i;
                    
                    // Actualizar progressBar via PrimeFaces
                    PrimeFaces.current().executeScript(
                        "updateProgress(" + progress + ");"
                    );
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                importing = false;
                PrimeFaces.current().executeScript("PF('importDialog').hide();");
            }
        }).start();
    }
    
    // Getters y setters
}

<!-- En la vista -->
<p:dialog header="Importando Datos" widgetVar="importDialog" modal="true">
    <h:form>
        <p:progressBar widgetVar="pb" value="#{importController.progress}" 
                       labelTemplate="{value}%"
                       styleClass="import-progress"
                       ajax="true">
            <p:ajax event="complete" listener="#{importController.onComplete}"
                    update=":messages" />
        </p:progressBar>
        
        <div class="progress-details">
            <p>Procesando registros... #{importController.progress}/100</p>
            <p:outputLabel value="Tiempo estimado: 2 minutos" />
        </div>
    </h:form>
</p:dialog>
```

# Solución Reutilizable: Loading Manager
5. LoadingManager Controller

```
package com.empresa.proyecto.util;

import org.primefaces.PrimeFaces;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import javax.inject.Named;
import java.io.Serializable;
import java.util.HashMap;
import java.util.Map;

/**
 * Gestor centralizado de loading indicators
 */
@Named
@ViewScoped
public class LoadingManager implements Serializable {
    
    private Map<String, Boolean> loadingStates;
    private Map<String, String> loadingMessages;
    
    @PostConstruct
    public void init() {
        loadingStates = new HashMap<>();
        loadingMessages = new HashMap<>();
    }
    
    /**
     * Iniciar loading para un componente específico
     */
    public void startLoading(String componentId) {
        startLoading(componentId, "Procesando...");
    }
    
    public void startLoading(String componentId, String message) {
        loadingStates.put(componentId, true);
        loadingMessages.put(componentId, message);
        
        // Mostrar loading visualmente
        PrimeFaces.current().executeScript(
            String.format("showLoading('%s', '%s');", componentId, message)
        );
    }
    
    /**
     * Detener loading para un componente
     */
    public void stopLoading(String componentId) {
        loadingStates.put(componentId, false);
        
        PrimeFaces.current().executeScript(
            String.format("hideLoading('%s');", componentId)
        );
    }
    
    /**
     * Verificar si un componente está en loading
     */
    public boolean isLoading(String componentId) {
        return loadingStates.getOrDefault(componentId, false);
    }
    
    /**
     * Loading para operaciones AJAX globales
     */
    public void showGlobalLoading() {
        PrimeFaces.current().executeScript("PF('globalLoading').show();");
    }
    
    public void hideGlobalLoading() {
        PrimeFaces.current().executeScript("PF('globalLoading').hide();");
    }
    
    /**
     * Loading con bloqueo de interfaz
     */
    public void blockUI(String target, String message) {
        String script = String.format(
            "PF('%sBlockUI').show(); PF('%sBlockUI').message('%s');",
            target, target, message
        );
        PrimeFaces.current().executeScript(script);
    }
    
    public void unblockUI(String target) {
        PrimeFaces.current().executeScript(
            String.format("PF('%sBlockUI').hide();", target)
        );
    }
}
```

 6. JavaScript Helper para Loading

```
// loading.js - Archivo reusable
var LoadingManager = {
    
    // Configuración
    config: {
        overlayColor: 'rgba(0, 0, 0, 0.5)',
        spinnerColor: '#2196F3',
        zIndex: 99999
    },
    
    // Spinners activos
    activeSpinners: {},
    
    /**
     * Mostrar loading overlay
     */
    showOverlay: function(targetId, message) {
        var target = $(PrimeFaces.escapeClientId(targetId));
        var overlayId = targetId + '_overlay';
        
        // Crear overlay si no existe
        if (!$('#' + overlayId).length) {
            var overlay = $('<div>')
                .attr('id', overlayId)
                .addClass('loading-overlay')
                .css({
                    'position': 'absolute',
                    'top': '0',
                    'left': '0',
                    'width': '100%',
                    'height': '100%',
                    'background': this.config.overlayColor,
                    'z-index': this.config.zIndex,
                    'display': 'flex',
                    'align-items': 'center',
                    'justify-content': 'center'
                });
            
            // Spinner
            var spinner = $('<div>')
                .addClass('spinner')
                .html(`
                    <div class="spinner-content">
                        <div class="spinner-border text-primary" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                        ${message ? '<p class="mt-2">' + message + '</p>' : ''}
                    </div>
                `);
            
            overlay.append(spinner);
            target.css('position', 'relative').append(overlay);
        }
        
        this.activeSpinners[overlayId] = true;
    },
    
    /**
     * Ocultar loading overlay
     */
    hideOverlay: function(targetId) {
        var overlayId = targetId + '_overlay';
        $('#' + overlayId).remove();
        delete this.activeSpinners[overlayId];
    },
    
    /**
     * Loading para botones
     */
    buttonLoading: function(buttonId, show) {
        var button = $(PrimeFaces.escapeClientId(buttonId));
        
        if (show) {
            // Guardar texto original
            button.data('original-text', button.html());
            
            // Mostrar spinner
            button.html(`
                <i class="pi pi-spin pi-spinner"></i>
                <span class="ms-2">Procesando...</span>
            `).prop('disabled', true);
        } else {
            // Restaurar texto original
            var original = button.data('original-text');
            if (original) {
                button.html(original);
            }
            button.prop('disabled', false);
        }
    },
    
    /**
     * Loading para tablas
     */
    tableLoading: function(tableId, show) {
        var table = $(PrimeFaces.escapeClientId(tableId));
        
        if (show) {
            table.addClass('table-loading');
            table.find('tbody').html(`
                <tr>
                    <td colspan="${table.find('thead th').length}" class="text-center">
                        <div class="table-spinner">
                            <i class="pi pi-spin pi-spinner fa-2x"></i>
                            <p>Cargando datos...</p>
                        </div>
                    </td>
                </tr>
            `);
        } else {
            table.removeClass('table-loading');
        }
    }
};

// Función global para usar desde XHTML
function showLoading(targetId, message) {
    LoadingManager.showOverlay(targetId, message);
}

function hideLoading(targetId) {
    LoadingManager.hideOverlay(targetId);
}
```
# Implementación en Botones CRUD
7. Botones con Loading Integrado

```
<!-- Botón con loading automático -->
<p:commandButton value="Guardar" 
                 action="#{usuarioController.save}"
                 styleClass="btn-with-loading"
                 onstart="LoadingManager.buttonLoading('#{clientId}', true)"
                 oncomplete="LoadingManager.buttonLoading('#{clientId}', false)"
                 onerror="LoadingManager.buttonLoading('#{clientId}', false)"
                 update="@form :messages"/>

<!-- Botón con callback para manejar respuesta -->
<p:commandButton value="Exportar Excel" 
                 action="#{reportController.exportToExcel}"
                 styleClass="export-btn"
                 onclick="showExportLoading()"
                 oncomplete="handleExportComplete(xhr, status, args)">
    <f:param name="reportType" value="detailed" />
</p:commandButton>

<script>
function showExportLoading() {
    // Bloquear UI
    PF('blockUIWidget').show();
    
    // Mostrar mensaje específico
    PF('blockUIWidget').message('Generando archivo Excel...');
    
    // Deshabilitar botones adicionales
    $('.export-btn').prop('disabled', true);
}

function handleExportComplete(xhr, status, args) {
    // Ocultar loading
    PF('blockUIWidget').hide();
    
    // Habilitar botones
    $('.export-btn').prop('disabled', false);
    
    if (args.success) {
        // Mostrar mensaje de éxito
        PF('growl').show([{
            severity: 'success',
            summary: 'Éxito',
            detail: 'Reporte exportado correctamente'
        }]);
        
        // Forzar descarga
        if (args.downloadUrl) {
            window.location.href = args.downloadUrl;
        }
    } else {
        // Mostrar error
        PF('growl').show([{
            severity: 'error',
            summary: 'Error',
            detail: args.errorMessage || 'Error al exportar'
        }]);
    }
}
</script>
```
8. Loading para Operaciones Largas (AJAX Polling)

```
<!-- Controller -->
@Named
@ViewScoped
public class BatchController implements Serializable {
    private String processId;
    private String processStatus;
    private Integer progress;
    
    public void startBatchProcess() {
        processId = UUID.randomUUID().toString();
        progress = 0;
        
        // Iniciar proceso asíncrono
        ExecutorService executor = Executors.newSingleThreadExecutor();
        executor.submit(() -> {
            try {
                // Simular proceso largo
                for (int i = 0; i <= 100; i += 10) {
                    Thread.sleep(2000);
                    progress = i;
                    
                    // Actualizar via PrimeFaces Push
                    PrimeFaces.current().push("/batchProgress", progress);
                }
                
                processStatus = "COMPLETED";
                PrimeFaces.current().push("/batchStatus", processStatus);
                
            } catch (Exception e) {
                processStatus = "FAILED";
                PrimeFaces.current().push("/batchStatus", processStatus);
            }
        });
    }
    
    // Getters y setters
}

<!-- Vista con polling -->
<p:dialog header="Proceso Batch" widgetVar="batchDialog">
    <h:form id="batchForm">
        <p:progressBar value="#{batchController.progress}" 
                       widgetVar="pb" ajax="true"/>
        
        <p:poll interval="3" 
                listener="#{batchController.checkStatus}"
                update="pb batchStatus"
                rendered="#{batchController.processId != null}"/>
        
        <p:outputLabel id="batchStatus" 
                      value="#{batchController.processStatus}"/>
        
        <p:commandButton value="Iniciar Proceso"
                         action="#{batchController.startBatchProcess}"
                         oncomplete="PF('batchDialog').show()"
                         update="batchForm"/>
    </h:form>
</p:dialog>
```

# CSS para Loading Personalizados

9. Estilos Reutilizables

```
/* loading.css - Estilos para loading indicators */

/* Overlay base */
.loading-overlay {
    background: rgba(255, 255, 255, 0.9) !important;
    backdrop-filter: blur(5px);
    z-index: 9999;
}

/* Spinner animations */
@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

.spinner-border {
    display: inline-block;
    width: 2rem;
    height: 2rem;
    vertical-align: text-bottom;
    border: 0.25em solid currentColor;
    border-right-color: transparent;
    border-radius: 50%;
    animation: spin 0.75s linear infinite;
}

/* Button loading states */
.btn-loading {
    position: relative;
}

.btn-loading .loading-spinner {
    display: inline-block;
    margin-right: 8px;
    animation: spin 1s linear infinite;
}

.btn-loading:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

/* Table loading */
.table-loading {
    position: relative;
    opacity: 0.7;
}

.table-spinner {
    padding: 40px;
    text-align: center;
    color: #666;
}

.table-spinner i {
    color: #2196F3;
    margin-bottom: 10px;
}

/* Skeleton loading (para contenido) */
.skeleton {
    background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
    background-size: 200% 100%;
    animation: loading 1.5s infinite;
    border-radius: 4px;
}

.skeleton-text {
    height: 12px;
    margin-bottom: 6px;
}

.skeleton-button {
    width: 100px;
    height: 36px;
}

@keyframes loading {
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
}

/* Progress bar custom */
.import-progress .ui-progressbar-value {
    background: linear-gradient(90deg, #4CAF50, #8BC34A);
    transition: width 0.3s ease;
}

.import-progress .ui-progressbar-label {
    color: white;
    font-weight: bold;
    text-shadow: 1px 1px 1px rgba(0,0,0,0.3);
}

/* Card loading */
.card-loading {
    position: relative;
    min-height: 200px;
}

.card-loading::after {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(255,255,255,0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
}

.card-loading::before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 40px;
    height: 40px;
    border: 4px solid #f3f3f3;
    border-top: 4px solid #3498db;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    z-index: 11;
}
```

#  Patrón Reusable: Componente Loading
10. Componente Composite Personalizado

```
<!-- resources/empresa/components/loading.xhtml -->
<ui:component xmlns="http://www.w3.org/1999/xhtml"
              xmlns:h="http://xmlns.jcp.org/jsf/html"
              xmlns:p="http://primefaces.org/ui"
              xmlns:cc="http://xmlns.jcp.org/jsf/composite">
    
    <cc:interface>
        <cc:attribute name="widgetVar" default="loadingComponent"/>
        <cc:attribute name="type" default="spinner"/> <!-- spinner, dots, bars -->
        <cc:attribute name="size" default="medium"/> <!-- small, medium, large -->
        <cc:attribute name="message" default="Cargando..."/>
        <cc:attribute name="overlay" default="true"/>
        <cc:attribute name="color" default="primary"/>
    </cc:interface>
    
    <cc:implementation>
        <div id="#{cc.clientId}" class="loading-component">
            <p:dialog widgetVar="#{cc.attrs.widgetVar}"
                      modal="#{cc.attrs.overlay}"
                      draggable="false"
                      closable="false"
                      resizable="false"
                      showHeader="false">
                
                <div class="loading-content #{cc.attrs.size} #{cc.attrs.type}">
                    <!-- Spinner type -->
                    <div class="spinner" rendered="#{cc.attrs.type == 'spinner'}">
                        <div class="spinner-circle #{cc.attrs.color}"></div>
                    </div>
                    
                    <!-- Dots type -->
                    <div class="dots" rendered="#{cc.attrs.type == 'dots'}">
                        <div class="dot dot1"></div>
                        <div class="dot dot2"></div>
                        <div class="dot dot3"></div>
                    </div>
                    
                    <!-- Message -->
                    <p class="loading-message">#{cc.attrs.message}</p>
                </div>
                
            </p:dialog>
        </div>
        
        <style>
            .loading-content {
                text-align: center;
                padding: 20px;
            }
            
            .spinner-circle {
                width: 40px;
                height: 40px;
                border: 4px solid #f3f3f3;
                border-top: 4px solid;
                border-radius: 50%;
                animation: spin 1s linear infinite;
                margin: 0 auto 15px;
            }
            
            .spinner-circle.primary { border-top-color: #2196F3; }
            .spinner-circle.success { border-top-color: #4CAF50; }
            .spinner-circle.warning { border-top-color: #FF9800; }
            .spinner-circle.danger { border-top-color: #F44336; }
            
            .dots {
                display: flex;
                justify-content: center;
                gap: 8px;
                margin-bottom: 15px;
            }
            
            .dot {
                width: 12px;
                height: 12px;
                border-radius: 50%;
                background: #2196F3;
                animation: bounce 1.4s infinite ease-in-out both;
            }
            
            .dot1 { animation-delay: -0.32s; }
            .dot2 { animation-delay: -0.16s; }
            
            @keyframes bounce {
                0%, 80%, 100% { transform: scale(0); }
                40% { transform: scale(1); }
            }
            
            .loading-message {
                margin: 0;
                color: #666;
                font-size: 14px;
            }
        </style>
    </cc:implementation>
</ui:component>
```
11. Uso del Componente

```
<!-- En cualquier página -->
<empresa:loading widgetVar="customLoading" 
                 type="dots" 
                 message="Procesando su solicitud..."
                 color="primary"/>

<!-- Botón que usa el componente -->
<p:commandButton value="Procesar"
                 action="#{controller.process}"
                 onstart="PF('customLoading').show()"
                 oncomplete="PF('customLoading').hide()"
                 update="@form"/>
```

# Ejemplos por tipo de operación:
```
<!-- Operación rápida (<2s) -->
<p:commandButton onstart="LoadingManager.buttonLoading(this)" .../>

<!-- Operación media (2-10s) -->
<p:commandButton onstart="PF('progressDialog').show()" .../>
<p:dialog widgetVar="progressDialog">
    <p:progressBar widgetVar="pb" ajax="true"/>
</p:dialog>

<!-- Operación larga (>10s) -->
<p:commandButton action="#{controller.startLongOperation}"
                 onstart="startLongOperationUI()"/>
                 
<script>
function startLongOperationUI() {
    PF('longOpDialog').show();
    PF('longOpProgress').start();
    // Polling para actualizar estado
    setInterval(updateProgress, 2000);
}
</script>
```

#  Guía de Calidad y Accesibilidad para Aplicaciones PrimeFaces

1. Estructura HTML Semántica Correcta

```
<!-- ✅ CORRECTO - Estructura semántica -->
<!DOCTYPE html>
<html lang="es" xmlns="http://www.w3.org/1999/xhtml">
<h:head>
    <!-- Título significativo y único por página -->
    <title>#{msgs['app.name']} - #{pageTitle} | #{sectionTitle}</title>
    
    <!-- Metadatos esenciales -->
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="description" content="#{pageDescription}" />
    <meta name="keywords" content="#{pageKeywords}" />
    <meta name="author" content="Empresa, S.A. de C.V." />
    
    <!-- Open Graph para redes sociales -->
    <meta property="og:title" content="#{pageTitle}" />
    <meta property="og:description" content="#{pageDescription}" />
    <meta property="og:image" content="#{resource['images/og-image.jpg']}" />
    
    <!-- Favicon en múltiples formatos -->
    <link rel="icon" type="image/x-icon" href="#{resource['images/favicon.ico']}" />
    <link rel="icon" type="image/png" sizes="32x32" href="#{resource['images/favicon-32x32.png']}" />
    <link rel="icon" type="image/png" sizes="16x16" href="#{resource['images/favicon-16x16.png']}" />
    <link rel="apple-touch-icon" sizes="180x180" href="#{resource['images/apple-touch-icon.png']}" />
    
    <!-- CSS con atributos correctos -->
    <h:outputStylesheet library="css" name="app.css" />
    <h:outputStylesheet library="css" name="print.css" media="print" />
    
    <!-- Preload de recursos críticos -->
    <link rel="preload" href="#{resource['css/critical.css']}" as="style" />
    
    <!-- Scripts diferidos -->
    <h:outputScript name="app.js" target="body" />
    
    <!-- ARIA landmark roles -->
    <f:passThroughAttributes>
        <f:passThroughAttribute name="role" value="banner" />
    </f:passThroughAttributes>
</h:head>

<h:body>
    <!-- Skip to main content link (accesibilidad) -->
    <a href="#main-content" class="skip-link">Saltar al contenido principal</a>
    
    <!-- Header con rol ARIA -->
    <header role="banner" class="layout-header">
        <!-- Logo con texto alternativo -->
        <h:graphicImage library="images" name="logo.png" 
                       alt="#{msgs['app.name']} - Logo" 
                       title="#{msgs['app.name']}" />
        
        <!-- Navegación principal -->
        <nav role="navigation" aria-label="Navegación principal">
            <!-- Lista semántica para navegación -->
            <ul>
                <li><p:link value="Inicio" outcome="/index" /></li>
                <!-- ... -->
            </ul>
        </nav>
    </header>
    
    <!-- Contenido principal -->
    <main id="main-content" role="main" tabindex="-1">
        <!-- Jerarquía de encabezados correcta -->
        <h1>#{pageTitle}</h1> <!-- Solo UN h1 por página -->
        
        <!-- Secciones con encabezados apropiados -->
        <section aria-labelledby="section1-title">
            <h2 id="section1-title">Título de Sección</h2>
            <!-- Contenido -->
        </section>
        
        <!-- Artículos independientes -->
        <article aria-labelledby="article1-title">
            <h3 id="article1-title">Título de Artículo</h3>
            <!-- Contenido -->
        </article>
    </main>
    
    <!-- Footer -->
    <footer role="contentinfo">
        <!-- Contenido del footer -->
    </footer>
</h:body>
</html>
```
2. Jerarquía Correcta de Encabezados (Headings)

```

<!-- ❌ INCORRECTO - Saltos en jerarquía -->
<h1>Página Principal</h1>
<h3>Subtítulo</h3> <!-- ¡Error! Debería ser h2 -->

<!-- ✅ CORRECTO - Jerarquía secuencial -->
<h1>#{pageController.pageTitle}</h1>
<h2>#{sectionController.sectionTitle}</h2>
<h3>#{subsectionController.subsectionTitle}</h3>

<!-- En componente reusable -->
<ui:composition>
    <h2 rendered="#{level == 2}">#{title}</h2>
    <h3 rendered="#{level == 3}">#{title}</h3>
    <h4 rendered="#{level == 4}">#{title}</h4>
</ui:composition>

<!-- Helper para encabezados dinámicos -->
<cc:interface>
    <cc:attribute name="level" type="java.lang.Integer" default="2" />
    <cc:attribute name="title" type="java.lang.String" required="true" />
    <cc:attribute name="ariaLabel" type="java.lang.String" />
</cc:interface>

<cc:implementation>
    <h2 rendered="#{cc.attrs.level == 2}" 
        aria-label="#{cc.attrs.ariaLabel}">
        #{cc.attrs.title}
    </h2>
    <h3 rendered="#{cc.attrs.level == 3}"
        aria-label="#{cc.attrs.ariaLabel}">
        #{cc.attrs.title}
    </h3>
    <!-- ... -->
</cc:implementation>
```

# Componentes PrimeFaces con Accesibilidad
5. Botones Accesibles

```
<!-- ❌ INCORRECTO -->
<p:commandButton value="Guardar" /> <!-- Falta type, aria-label -->

<!-- ✅ CORRECTO -->
<p:commandButton value="Guardar"
                 type="submit"
                 aria-label="Guardar cambios"
                 title="Haz clic para guardar los cambios"
                 icon="pi pi-save"
                 styleClass="p-button-primary" />

<!-- Para iconos sin texto -->
<p:commandButton icon="pi pi-trash"
                 aria-label="Eliminar registro"
                 title="Eliminar este registro"
                 onclick="return confirm('¿Está seguro?')" />

<!-- Grupo de botones con roles ARIA -->
<div role="toolbar" aria-label="Herramientas de edición">
    <p:commandButton icon="pi pi-pencil"
                     aria-label="Editar"
                     title="Editar registro" />
    <p:commandButton icon="pi pi-copy"
                     aria-label="Duplicar"
                     title="Duplicar registro" />
</div>

```
# Validación Completa de Formularios en PrimeFaces
Validación Frontend (XHTML/JavaScript)
1. Validaciones Básicas con JSF

```
<!-- Campo requerido -->
<p:inputText id="nombre" 
             value="#{bean.nombre}"
             required="true"
             requiredMessage="El nombre es obligatorio">
</p:inputText>
<p:message for="nombre" />

<!-- Longitud mínima y máxima -->
<p:inputText id="codigo"
             value="#{bean.codigo}"
             required="true">
    <f:validateLength minimum="3" maximum="10" />
</p:inputText>

<!-- Expresión regular -->
<p:inputText id="email"
             value="#{bean.email}">
    <f:validateRegex pattern="^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$" />
</p:inputText>

<!-- Rango numérico -->
<p:inputText id="edad"
             value="#{bean.edad}">
    <f:validateLongRange minimum="18" maximum="100" />
</p:inputText>

<!-- Validación personalizada -->
<p:inputText id="codigoPostal"
             value="#{bean.codigoPostal}"
             validator="#{bean.validarCodigoPostal}">
</p:inputText>
```
2. Campos Numéricos (Solo Números)
```
<!-- Input solo números enteros -->
<p:inputText id="cantidad"
             value="#{bean.cantidad}">
    <f:validateLongRange />
    <p:keyFilter mask="num" /> <!-- Solo números -->
</p:inputText>

<!-- Input números decimales -->
<p:inputText id="precio"
             value="#{bean.precio}">
    <f:validateDoubleRange minimum="0.0" maximum="999999.99" />
    <p:keyFilter mask="num" /> 
</p:inputText>

<!-- Input con formato monetario -->
<p:inputNumber id="monto"
               value="#{bean.monto}"
               decimalSeparator="."
               thousandSeparator=","
               symbol="$"
               symbolPosition="l"
               minValue="0"
               maxValue="1000000">
</p:inputNumber>

<!-- Spinner numérico -->
<p:spinner id="unidades"
           value="#{bean.unidades}"
           min="0"
           max="100"
           step="1">
</p:spinner>

```
3. Campos de Texto con Validaciones Específicas
```
<!-- Solo letras y espacios -->
<p:inputText id="nombreCompleto"
             value="#{bean.nombreCompleto}">
    <p:keyFilter mask="pnum" /> <!-- letras y números -->
</p:inputText>

<!-- Solo letras -->
<p:inputText id="primerNombre"
             value="#{bean.primerNombre}">
    <p:keyFilter mask="alpha" /> <!-- solo letras -->
</p:inputText>

<!-- Alfanumérico sin espacios -->
<p:inputText id="username"
             value="#{bean.username}">
    <p:keyFilter mask="alphanum" /> <!-- alfanumérico -->
</p:inputText>
```
4. Máscaras de Formato (InputMask)
```
<!-- Teléfono con formato -->
<p:inputMask id="telefono"
             value="#{bean.telefono}"
             mask="(999) 999-9999"
             placeholder="(555) 123-4567">
</p:inputMask>

<!-- Fecha -->
<p:inputMask id="fecha"
             value="#{bean.fecha}"
             mask="99/99/9999"
             placeholder="dd/mm/aaaa">
</p:inputMask>

<!-- RFC mexicano -->
<p:inputMask id="rfc"
             value="#{bean.rfc}"
             mask="aaaa999999aaa"
             placeholder="ABCD123456XYZ">
</p:inputMask>

<!-- CURP mexicana -->
<p:inputMask id="curp"
             value="#{bean.curp}"
             mask="aaaa999999aaaaaa99"
             placeholder="ABCD123456XYZLMN01">
</p:inputMask>

<!-- Tarjeta de crédito -->
<p:inputMask id="tarjetaCredito"
             value="#{bean.tarjetaCredito}"
             mask="9999-9999-9999-9999">
</p:inputMask>
```
5. Validadores Personalizados (Frontend)
```
// Validaciones JavaScript personalizadas
function validarFormulario() {
    var valido = true;
    
    // Validar RFC mexicano
    valido = valido && validarRFC(document.getElementById('form:rfc').value);
    
    // Validar CURP mexicana
    valido = valido && validarCURP(document.getElementById('form:curp').value);
    
    // Validar código postal mexicano
    valido = valido && validarCP(document.getElementById('form:cp').value);
    
    return valido;
}

// Funciones de validación específicas para México
function validarRFC(rfc) {
    var regex = /^[A-ZÑ&]{3,4}\d{6}[A-Z0-9]{2}[0-9A]$/;
    return regex.test(rfc);
}

function validarCURP(curp) {
    var regex = /^[A-Z]{4}\d{6}[HM]{1}[A-Z]{5}[A-Z0-9]{2}$/;
    return regex.test(curp);
}

function validarCP(cp) {
    return /^\d{5}$/.test(cp);
}
```
6. Validación Completa de Formulario con PrimeFaces

 ```
<h:form id="usuarioForm">
    
    <!-- Mensajes de validación agrupados -->
    <p:messages id="messages" showDetail="true" closable="true" />
    
    <!-- Resumen de errores (accesibilidad) -->
    <div role="alert" aria-live="assertive" class="error-summary"
         rendered="#{facesContext.validationFailed}">
        <h3>Corrige los siguientes errores:</h3>
        <ul>
            <ui:repeat value="#{facesContext.messageList}" var="msg">
                <li><a href="##{msg.clientId}">#{msg.summary}</a></li>
            </ui:repeat>
        </ul>
    </div>
    
    <p:panelGrid columns="3" styleClass="form-grid">
        
        <!-- Nombre (solo letras y espacios) -->
        <p:outputLabel for="nombre" value="Nombre *" />
        <p:inputText id="nombre"
                     value="#{usuarioController.usuario.nombre}"
                     required="true"
                     requiredMessage="El nombre es obligatorio"
                     maxlength="100">
            <p:keyFilter mask="alpha" /> <!-- solo letras -->
            <f:validateLength minimum="2" maximum="100" />
            <p:ajax event="blur" update="nombreMsg" />
        </p:inputText>
        <p:message for="nombre" id="nombreMsg" />
        
        <!-- Email con formato -->
        <p:outputLabel for="email" value="Email *" />
        <p:inputText id="email"
                     value="#{usuarioController.usuario.email}"
                     required="true"
                     requiredMessage="Email requerido">
            <f:validateRegex pattern="^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$" />
            <p:ajax event="blur" 
                    listener="#{usuarioController.validarEmailUnico}"
                    update="emailMsg" />
        </p:inputText>
        <p:message for="email" id="emailMsg" />
        
        <!-- Teléfono mexicano -->
        <p:outputLabel for="telefono" value="Teléfono *" />
        <p:inputMask id="telefono"
                     value="#{usuarioController.usuario.telefono}"
                     mask="(999) 999-9999"
                     required="true">
            <p:ajax event="change" update="telefonoMsg" />
        </p:inputMask>
        <p:message for="telefono" id="telefonoMsg" />
        
        <!-- RFC mexicano -->
        <p:outputLabel for="rfc" value="RFC *" />
        <p:inputMask id="rfc"
                     value="#{usuarioController.usuario.rfc}"
                     mask="aaaa999999aaa"
                     required="true">
            <p:ajax event="change" 
                    listener="#{usuarioController.validarRFC}"
                    update="rfcMsg" />
        </p:inputMask>
        <p:message for="rfc" id="rfcMsg" />
        
        <!-- Edad (solo números) -->
        <p:outputLabel for="edad" value="Edad *" />
        <p:spinner id="edad"
                   value="#{usuarioController.usuario.edad}"
                   min="18"
                   max="100"
                   required="true">
            <p:ajax event="change" update="edadMsg" />
        </p:spinner>
        <p:message for="edad" id="edadMsg" />
        
        <!-- Salario (decimal con formato) -->
        <p:outputLabel for="salario" value="Salario *" />
        <p:inputNumber id="salario"
                       value="#{usuarioController.usuario.salario}"
                       decimalSeparator="."
                       thousandSeparator=","
                       symbol="$"
                       minValue="0"
                       maxValue="999999.99"
                       required="true">
            <p:ajax event="change" update="salarioMsg" />
        </p:inputNumber>
        <p:message for="salario" id="salarioMsg" />
        
    </p:panelGrid>
    
    <!-- Botón con validación completa -->
    <p:commandButton value="Guardar"
                     action="#{usuarioController.guardar}"
                     update="@form :growl"
                     process="@form"
                     onclick="return validarFormularioCompleto()"
                     onstart="PF('loading').show()"
                     oncomplete="PF('loading').hide()" />
    
</h:form>

<script>
function validarFormularioCompleto() {
    // Validaciones adicionales en JavaScript
    var email = document.getElementById('usuarioForm:email').value;
    if (!validarEmail(email)) {
        alert('Formato de email inválido');
        return false;
    }
    
    var rfc = document.getElementById('usuarioForm:rfc').value;
    if (!validarRFCMexicano(rfc)) {
        alert('RFC inválido');
        return false;
    }
    
    return true;
}
</script>

   ```
# Validación Backend (Java/Spring)

7. DTOs con Bean Validation (JSR 380)
  ```
package com.empresa.proyecto.dto;

import javax.validation.constraints.*;
import java.math.BigDecimal;
import java.time.LocalDate;

public class UsuarioDTO implements Serializable {
    
    @NotNull(message = "El nombre no puede ser nulo")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    @Pattern(regexp = "^[a-zA-ZáéíóúÁÉÍÓÚñÑ ]+$", 
             message = "El nombre solo puede contener letras y espacios")
    private String nombre;
    
    @NotNull(message = "El email no puede ser nulo")
    @Email(message = "Formato de email inválido")
    @Column(unique = true)
    private String email;
    
    @NotNull(message = "El teléfono no puede ser nulo")
    @Pattern(regexp = "^\\d{10}$", 
             message = "El teléfono debe tener 10 dígitos")
    private String telefono;
    
    @NotNull(message = "El RFC no puede ser nulo")
    @Pattern(regexp = "^[A-ZÑ&]{3,4}\\d{6}[A-Z0-9]{2}[0-9A]$", 
             message = "Formato de RFC inválido")
    private String rfc;
    
    @NotNull(message = "La edad no puede ser nula")
    @Min(value = 18, message = "La edad mínima es 18 años")
    @Max(value = 100, message = "La edad máxima es 100 años")
    private Integer edad;
    
    @NotNull(message = "El salario no puede ser nulo")
    @DecimalMin(value = "0.0", inclusive = false, 
                message = "El salario debe ser mayor a 0")
    @DecimalMax(value = "999999.99", 
                message = "El salario máximo es 999,999.99")
    @Digits(integer = 6, fraction = 2, 
            message = "El salario debe tener máximo 6 enteros y 2 decimales")
    private BigDecimal salario;
    
    @NotNull(message = "La fecha de nacimiento no puede ser nula")
    @Past(message = "La fecha de nacimiento debe ser en el pasado")
    private LocalDate fechaNacimiento;
    
    @NotNull(message = "La contraseña no puede ser nula")
    @Size(min = 8, max = 20, 
          message = "La contraseña debe tener entre 8 y 20 caracteres")
    @Pattern(regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$", 
             message = "La contraseña debe contener al menos un número, una letra minúscula, una mayúscula y un carácter especial")
    private String password;
    
    @AssertTrue(message = "Debe aceptar los términos y condiciones")
    private boolean aceptaTerminos;
    
    // Getters y Setters
    // ... (con validaciones adicionales si es necesario)
    
    public void setEmail(String email) {
        if (email != null) {
            this.email = email.toLowerCase().trim();
        }
    }
    
    public void setTelefono(String telefono) {
        if (telefono != null) {
            this.telefono = telefono.replaceAll("[^0-9]", "");
        }
    }
}
  ```

8. Validadores Personalizados (Backend)
 ```
package com.empresa.proyecto.validator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = RFCValidator.class)
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface RFCValido {
    String message() default "RFC inválido";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Implementación del validador
@Component
public class RFCValidator implements ConstraintValidator<RFCValido, String> {
    
    private static final String RFC_REGEX = 
        "^[A-ZÑ&]{3,4}\\d{6}[A-Z0-9]{2}[0-9A]$";
    
    @Override
    public boolean isValid(String rfc, ConstraintValidatorContext context) {
        if (rfc == null || rfc.trim().isEmpty()) {
            return true; // Usar @NotNull para requerido
        }
        
        // Validar formato básico
        if (!rfc.matches(RFC_REGEX)) {
            return false;
        }
        
        // Validación más específica (homoclave, etc.)
        return validarRFCMexicano(rfc);
    }
    
    private boolean validarRFCMexicano(String rfc) {
        // Lógica específica de validación de RFC mexicano
        // Verificar homoclave, checksum, etc.
        return true; // Simplificado para ejemplo
    }
}

// Uso en DTO
public class UsuarioDTO {
    @RFCValido
    private String rfc;
}
 ```

9. Validación en Service Layer
 ```
@Service
@Transactional
@Slf4j
public class UsuarioService {
    
    @Autowired
    private UsuarioRepository usuarioRepository;
    
    @Autowired
    private Validator validator; // Inyectar validador JSR-380
    
    public UsuarioDTO crearUsuario(@Valid UsuarioDTO usuarioDTO) {
        // Validación JSR-380 automática
        // Si hay errores, se lanza ConstraintViolationException
        
        // Validaciones de negocio adicionales
        validarNegocio(usuarioDTO);
        
        // Convertir DTO a Entity
        Usuario usuario = convertirAEntity(usuarioDTO);
        
        // Guardar
        usuario = usuarioRepository.save(usuario);
        
        // Convertir de vuelta a DTO
        return convertirADTO(usuario);
    }
    
    private void validarNegocio(UsuarioDTO usuarioDTO) {
        // Validar que el email no exista
        if (usuarioRepository.existsByEmail(usuarioDTO.getEmail())) {
            throw new BusinessValidationException(
                "El email " + usuarioDTO.getEmail() + " ya está registrado");
        }
        
        // Validar que el RFC no exista
        if (usuarioRepository.existsByRfc(usuarioDTO.getRfc())) {
            throw new BusinessValidationException(
                "El RFC " + usuarioDTO.getRfc() + " ya está registrado");
        }
        
        // Validar edad mínima para ciertos productos
        if (usuarioDTO.getEdad() < 18 && usuarioDTO.getProductoRiesgoso()) {
            throw new BusinessValidationException(
                "Debe ser mayor de 18 años para este producto");
        }
        
        // Validaciones específicas de México
        validarDatosMexicanos(usuarioDTO);
    }
    
    private void validarDatosMexicanos(UsuarioDTO usuarioDTO) {
        // Validar formato de CURP si se proporciona
        if (usuarioDTO.getCurp() != null) {
            if (!validarCURP(usuarioDTO.getCurp())) {
                throw new BusinessValidationException("CURP inválida");
            }
        }
        
        // Validar código postal mexicano
        if (!validarCodigoPostalMX(usuarioDTO.getCodigoPostal())) {
            throw new BusinessValidationException(
                "Código Postal inválido para México");
        }
        
        // Validar estado de la república
        if (!validarEstadoMX(usuarioDTO.getEstado())) {
            throw new BusinessValidationException("Estado inválido");
        }
    }
    
    // Métodos de validación específicos
    private boolean validarCURP(String curp) {
        String regex = "^[A-Z]{4}\\d{6}[HM]{1}[A-Z]{5}[A-Z0-9]{2}$";
        return curp.matches(regex);
    }
    
    private boolean validarCodigoPostalMX(String cp) {
        return cp != null && cp.matches("^\\d{5}$");
    }
    
    private boolean validarEstadoMX(String estado) {
        List<String> estados = Arrays.asList(
            "AGU", "BCN", "BCS", "CAM", "CHP", "CHH", "COA", "COL",
            "DUR", "GUA", "GRO", "HID", "JAL", "MEX", "MIC", "MOR",
            "NAY", "NLE", "OAX", "PUE", "QUE", "ROO", "SLP", "SIN",
            "SON", "TAB", "TAM", "TLA", "VER", "YUC", "ZAC"
        );
        return estados.contains(estado);
    }
}
 ```
10. Controller con Validación
 ```
@Named
@ViewScoped
@Slf4j
public class UsuarioController {
    
    @Inject
    private UsuarioService usuarioService;
    
    @Inject
    private Validator validator;
    
    private UsuarioDTO usuario = new UsuarioDTO();
    
    public void guardar() {
        try {
            // Validar DTO
            Set<ConstraintViolation<UsuarioDTO>> violations = 
                validator.validate(usuario);
            
            if (!violations.isEmpty()) {
                // Mostrar errores de validación
                mostrarErroresValidacion(violations);
                return;
            }
            
            // Guardar usuario
            usuarioService.crearUsuario(usuario);
            
            // Mensaje de éxito
            FacesUtil.addSuccessMessage("Usuario creado exitosamente");
            
            // Resetear formulario
            resetForm();
            
        } catch (BusinessValidationException e) {
            // Errores de negocio
            FacesUtil.addErrorMessage(e.getMessage());
            log.warn("Error de validación de negocio: {}", e.getMessage());
            
        } catch (ConstraintViolationException e) {
            // Errores de Bean Validation
            mostrarErroresValidacion(e.getConstraintViolations());
            
        } catch (Exception e) {
            // Error general
            FacesUtil.addErrorMessage("Error al guardar usuario");
            log.error("Error al guardar usuario", e);
        }
    }
    
    private void mostrarErroresValidacion(
            Set<ConstraintViolation<UsuarioDTO>> violations) {
        
        for (ConstraintViolation<UsuarioDTO> violation : violations) {
            String campo = violation.getPropertyPath().toString();
            String mensaje = violation.getMessage();
            
            FacesUtil.addErrorMessage(
                String.format("%s: %s", campo, mensaje)
            );
        }
    }
    
    // Validación en tiempo real (AJAX)
    public void validarEmailUnico() {
        if (usuario.getEmail() != null && !usuario.getEmail().isEmpty()) {
            boolean existe = usuarioService.existeEmail(usuario.getEmail());
            if (existe) {
                FacesUtil.addErrorMessage("El email ya está registrado");
            }
        }
    }
    
    public void validarRFC() {
        if (usuario.getRfc() != null && !usuario.getRfc().isEmpty()) {
            // Validar formato RFC
            if (!usuario.getRfc().matches(
                "^[A-ZÑ&]{3,4}\\d{6}[A-Z0-9]{2}[0-9A]$")) {
                
                FacesUtil.addErrorMessage("Formato de RFC inválido");
            }
        }
    }
    
    private void resetForm() {
        usuario = new UsuarioDTO();
    }
}
 ```


5. Formularios Accesibles

```
<!-- ❌ INCORRECTO -->
<p:inputText value="#{bean.value}" />

<!-- ✅ CORRECTO -->
<p:outputLabel for="nombre" value="Nombre completo *" />
<p:inputText id="nombre"
             value="#{usuarioController.usuario.nombre}"
             required="true"
             aria-required="true"
             aria-describedby="nombreHelp nombreError"
             placeholder="Ingrese su nombre completo"
             maxlength="100" />
<p:message for="nombre" id="nombreError" />
<small id="nombreHelp" class="help-text">
    Ingrese su nombre completo como aparece en su identificación oficial.
</small>

<!-- Fieldset para grupos -->
<fieldset aria-labelledby="direccion-legend">
    <legend id="direccion-legend">Información de Dirección</legend>
    
    <p:outputLabel for="calle" value="Calle *" />
    <p:inputText id="calle" value="#{direccion.calle}" 
                 required="true" />
</fieldset>
```

7. Tablas Accesibles

```
<!--  INCORRECTO -->
<p:dataTable value="#{list}" var="item">
    <p:column headerText="Nombre">
        #{item.nombre}
    </p:column>
</p:dataTable>

<!-- CORRECTO -->
<p:dataTable value="#{usuarioController.usuarios}" 
             var="usuario"
             aria-label="Lista de usuarios registrados"
             summary="Tabla que muestra los usuarios del sistema con sus nombres, emails y estados"
             rowIndexVar="rowIndex"
             paginator="true"
             rows="10">
    
    <!-- Columnas con scope y headers -->
    <p:column headerText="#" 
              style="width: 50px;"
              aria-label="Número de fila">
        #{rowIndex + 1}
    </p:column>
    
    <p:column headerText="Nombre completo"
              sortBy="#{usuario.nombreCompleto}"
              filterBy="#{usuario.nombreCompleto}"
              aria-sort="none">
        <h:outputText value="#{usuario.nombreCompleto}"
                      title="Nombre completo del usuario" />
    </p:column>
    
    <p:column headerText="Email"
              aria-describedby="email-desc">
        <h:outputText value="#{usuario.email}" />
        <span id="email-desc" class="visually-hidden">
            Dirección de correo electrónico
        </span>
    </p:column>
    
    <!-- Acciones accesibles -->
    <p:column headerText="Acciones"
              aria-label="Acciones disponibles">
        <div role="group" aria-label="Acciones para usuario #{usuario.nombre}">
            <p:commandButton icon="pi pi-eye"
                             aria-label="Ver detalles del usuario #{usuario.nombre}"
                             title="Ver detalles" />
            <p:commandButton icon="pi pi-pencil"
                             aria-label="Editar usuario #{usuario.nombre}"
                             title="Editar" />
        </div>
    </p:column>
</p:dataTable>
```

#  Plantillas Reusables con Accesibilidad

```
<!-- base-page.xhtml -->
<ui:composition xmlns="http://www.w3.org/1999/xhtml"
                xmlns:h="http://xmlns.jcp.org/jsf/html"
                xmlns:p="http://primefaces.org/ui"
                xmlns:f="http://xmlns.jcp.org/jsf/core"
                xmlns:ui="http://xmlns.jcp.org/jsf/facelets"
                template="/WEB-INF/templates/template.xhtml">

    <f:metadata>
        <!-- Metadata dinámica para SEO -->
        <f:viewParam name="pageTitle" value="#{pageController.pageTitle}" />
        <f:viewParam name="pageDescription" value="#{pageController.pageDescription}" />
        <f:viewAction action="#{pageController.initPage}" />
    </f:metadata>

    <ui:define name="metadata">
        <!-- Metadata adicional específica de página -->
        <meta name="robots" content="#{pageController.robotsMeta}" />
        <meta property="og:type" content="#{pageController.ogType}" />
        <link rel="canonical" href="#{pageController.canonicalUrl}" />
    </ui:define>

    <ui:define name="content">
        <!-- Contenedor principal con ARIA -->
        <main id="main-content" 
              role="main" 
              class="page-content #{pageController.pageClass}"
              aria-labelledby="page-title">
            
            <!-- Título de página (solo un h1) -->
            <h1 id="page-title" class="page-title">
                #{pageController.pageTitle}
            </h1>
            
            <!-- Breadcrumbs accesibles -->
            <nav aria-label="Miga de pan" class="breadcrumb">
                <ol>
                    <li><a href="#{request.contextPath}/">Inicio</a></li>
                    <ui:repeat value="#{pageController.breadcrumbs}" var="crumb">
                        <li>
                            <a href="#{crumb.url}" 
                               aria-current="#{crumb.current ? 'page' : 'false'}">
                                #{crumb.label}
                            </a>
                        </li>
                    </ui:repeat>
                </ol>
            </nav>
            
            <!-- Contenido específico de la página -->
            <ui:insert name="page-content">
                <!-- Será reemplazado por páginas hijas -->
            </ui:insert>
            
        </main>
        
        <!-- Navegación lateral (si aplica) -->
        <aside role="complementary" 
               aria-labelledby="sidebar-title"
               rendered="#{pageController.hasSidebar}">
            <h2 id="sidebar-title" class="visually-hidden">
                Navegación secundaria
            </h2>
            <ui:insert name="sidebar" />
        </aside>
        
    </ui:define>
    
    <!-- Scripts específicos de página -->
    <ui:define name="scripts">
        <h:outputScript library="js" name="page-specific.js" />
    </ui:define>
    
</ui:composition>
```

3. Componente de Formulario Accesible

```
<!-- accessible-form.xhtml -->
<cc:interface>
    <cc:attribute name="bean" required="true" />
    <cc:attribute name="action" required="true" />
    <cc:attribute name="legend" required="true" />
    <cc:attribute name="ariaDescribedby" />
    <cc:attribute name="validate" default="true" />
</cc:interface>

<cc:implementation>
    <fieldset aria-labelledby="#{cc.clientId}-legend" 
              aria-describedby="#{cc.attrs.ariaDescribedby}">
        
        <legend id="#{cc.clientId}-legend">
            #{cc.attrs.legend}
        </legend>
        
        <div class="form-grid" role="form">
            
            <!-- Campo de texto con validación -->
            <cc:insertFacet name="field" />
            
            <!-- Mensajes de error agrupados -->
            <div role="alert" 
                 aria-live="polite"
                 class="error-summary"
                 rendered="#{facesContext.validationFailed}">
                <h3>Corrige los siguientes errores:</h3>
                <ul>
                    <ui:repeat value="#{facesContext.messageList}" var="msg">
                        <li>
                            <a href="#{msg.detail}">
                                #{msg.summary}
                            </a>
                        </li>
                    </ui:repeat>
                </ul>
            </div>
            
            <!-- Botones de acción -->
            <div class="form-actions" role="group" aria-label="Acciones del formulario">
                <p:commandButton value="Guardar"
                                 action="#{cc.attrs.action}"
                                 type="submit"
                                 aria-label="Guardar cambios"
                                 icon="pi pi-save"
                                 styleClass="p-button-primary" />
                                 
                <p:button value="Cancelar"
                          outcome="back"
                          aria-label="Cancelar y volver"
                          icon="pi pi-times"
                          styleClass="p-button-secondary" />
            </div>
            
        </div>
        
    </fieldset>
</cc:implementation>
 ```
# SOPORTE Y MANTENIMIENTO
Estructura de Logging:

 ```
@Slf4j
@Named
public class UserController {
    public void saveUser() {
        try {
            log.info("Iniciando guardado de usuario");
            userService.save(user);
            log.info("Usuario guardado exitosamente: {}", user.getId());
        } catch (Exception e) {
            log.error("Error al guardar usuario", e);
            throw e;
        }
    }
}
 ```
1. Logger nativo de Java (java.util.logging)
```
import java.util.logging.Logger;
import java.util.logging.Level;

public class UsuarioService {
    // Crear logger por clase
    private static final Logger logger = 
        Logger.getLogger(UsuarioService.class.getName());
    
    public void metodoEjemplo() {
        logger.info("Iniciando proceso...");
        try {
            // código
            logger.fine("Detalle específico del proceso");
        } catch (Exception e) {
            logger.log(Level.SEVERE, "Error en el proceso", e);
        }
    }
}
```
2. Log4j 2 (Recomendado para producción)

```
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.20.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.20.0</version>
</dependency>
```

```
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class UsuarioService {
    // Logger estático por clase (patrón recomendado)
    private static final Logger logger = 
        LogManager.getLogger(UsuarioService.class);
    
    // O también puedes usar:
    // private static final Logger logger = LogManager.getLogger();
    
    public void procesarUsuario() {
        // Niveles de logging
        logger.trace("Mensaje de trace - máximo detalle");
        logger.debug("Mensaje de debug - información de desarrollo");
        logger.info("Mensaje de info - información general");
        logger.warn("Mensaje de warning - advertencia");
        logger.error("Mensaje de error - error recuperable");
        logger.fatal("Mensaje de fatal - error crítico");
    }
}
```

# Niveles de Logging y Cuándo Usarlos
Jerarquía de Niveles (de más a menos detalle):
```
TRACE > DEBUG > INFO > WARN > ERROR > FATAL
```
# Guía de Uso por Nivel:
```
public class LoggingEjemplo {
    private static final Logger logger = 
        LoggerFactory.getLogger(LoggingEjemplo.class);
    
    public void ejemploCompleto() {
        // 1. TRACE - Para trazar flujo detallado (solo desarrollo)
        logger.trace("Entrando al método procesarFactura()");
        logger.trace("Parámetros recibidos: facturaId={}, usuario={}", 
                     facturaId, usuario);
        
        // 2. DEBUG - Información para debugging (desarrollo/testing)
        logger.debug("Buscando factura ID: {}", facturaId);
        logger.debug("Usuario tiene permisos: {}", tienePermisos);
        
        // 3. INFO - Información general de operación (producción)
        logger.info("Procesando factura {} para cliente {}", 
                    facturaId, clienteNombre);
        logger.info("Factura {} procesada exitosamente", facturaId);
        
        // 4. WARN - Situaciones inesperadas pero manejables
        if (factura.getMonto() > LIMITE) {
            logger.warn("Factura {} excede límite: {} > {}", 
                       facturaId, factura.getMonto(), LIMITE);
        }
        
        // 5. ERROR - Errores que requieren atención pero no caen la app
        try {
            procesarPago(factura);
        } catch (PaymentException e) {
            logger.error("Error procesando pago para factura {}", 
                        facturaId, e);
            // Continuar con otro método de pago
        }
        
        // 6. FATAL/ERROR crítico - Errores que hacen caer la aplicación
        try {
            inicializarSistema();
        } catch (InitializationException e) {
            logger.error("Error crítico iniciando sistema", e);
            System.exit(1); // Salir de la aplicación
        }
    }
}
```
# Implementación Correcta por Capa
4. Controller Layer (PrimeFaces)
```
@Named
@ViewScoped
public class UsuarioController {
    private static final Logger logger = 
        LoggerFactory.getLogger(UsuarioController.class);
    
    @Inject
    private UsuarioService usuarioService;
    
    public void guardarUsuario() {
        try {
            logger.info("Iniciando guardado de usuario - IP: {}", 
                       obtenerIPCliente());
            logger.debug("Datos usuario: {}", usuario.toString());
            
            usuarioService.guardar(usuario);
            
            logger.info("Usuario {} guardado exitosamente", usuario.getId());
            FacesUtil.addSuccessMessage("Usuario guardado");
            
        } catch (ValidacionException e) {
            // Error de validación de negocio
            logger.warn("Validación fallida para usuario {}: {}", 
                       usuario.getEmail(), e.getMessage());
            FacesUtil.addErrorMessage(e.getMessage());
            
        } catch (DuplicadoException e) {
            // Recurso duplicado
            logger.warn("Intento de crear usuario duplicado: {}", 
                       usuario.getEmail());
            FacesUtil.addErrorMessage("El usuario ya existe");
            
        } catch (Exception e) {
            // Error inesperado
            logger.error("Error inesperado guardando usuario {}", 
                        usuario.getEmail(), e);
            FacesUtil.addErrorMessage("Error interno del sistema");
        }
    }
    
    public void cargarUsuario(Long id) {
        try {
            logger.debug("Cargando usuario ID: {}", id);
            usuario = usuarioService.buscarPorId(id);
            
            if (usuario == null) {
                logger.warn("Usuario {} no encontrado", id);
                FacesUtil.addWarnMessage("Usuario no encontrado");
            } else {
                logger.debug("Usuario {} cargado: {}", id, usuario.getNombre());
            }
            
        } catch (Exception e) {
            logger.error("Error cargando usuario {}", id, e);
            FacesUtil.addErrorMessage("Error cargando usuario");
        }
    }
}
```
5. Service Layer (Lógica de Negocio)
```
@Service
@Transactional
public class UsuarioService {
    private static final Logger logger = 
        LoggerFactory.getLogger(UsuarioService.class);
    
    @Inject
    private UsuarioRepository usuarioRepository;
    
    @Inject
    private EmailService emailService;
    
    public Usuario guardar(Usuario usuario) {
        // Inicio de operación
        logger.info("Iniciando guardado de usuario: {}", usuario.getEmail());
        
        try {
            // Validación de negocio
            validarUsuario(usuario);
            
            // Operación principal
            logger.debug("Persistiendo usuario en BD");
            usuario = usuarioRepository.save(usuario);
            
            // Operación secundaria
            logger.debug("Enviando email de confirmación");
            emailService.enviarConfirmacionRegistro(usuario);
            
            logger.info("Usuario {} guardado exitosamente, ID: {}", 
                       usuario.getEmail(), usuario.getId());
            
            return usuario;
            
        } catch (ValidacionException e) {
            logger.warn("Validación fallida para usuario {}: {}", 
                       usuario.getEmail(), e.getMessage());
            throw e;
            
        } catch (EmailException e) {
            // Error en operación secundaria - no revertir transacción
            logger.error("Error enviando email para usuario {}, pero usuario guardado: {}", 
                        usuario.getEmail(), e.getMessage());
            // Continuar sin lanzar excepción
            return usuario;
            
        } catch (Exception e) {
            logger.error("Error crítico guardando usuario {}", 
                        usuario.getEmail(), e);
            throw new ServiceException("Error guardando usuario", e);
        }
    }
    
    public Page<Usuario> buscarUsuarios(UsuarioFiltro filtro, Pageable pageable) {
        logger.debug("Buscando usuarios con filtro: {}, página: {}", 
                    filtro, pageable.getPageNumber());
        
        Page<Usuario> resultado = usuarioRepository.findByFiltro(filtro, pageable);
        
        logger.info("Encontrados {} usuarios (página {} de {})", 
                   resultado.getTotalElements(),
                   resultado.getNumber(),
                   resultado.getTotalPages());
        
        return resultado;
    }
    
    private void validarUsuario(Usuario usuario) {
        logger.trace("Validando usuario: {}", usuario);
        
        if (usuarioRepository.existsByEmail(usuario.getEmail())) {
            logger.warn("Email duplicado detectado: {}", usuario.getEmail());
            throw new DuplicadoException("Email ya registrado");
        }
        
        if (!validarRFC(usuario.getRfc())) {
            logger.warn("RFC inválido: {}", usuario.getRfc());
            throw new ValidacionException("RFC inválido");
        }
        
        logger.trace("Usuario validado exitosamente");
    }
}
```
6. Repository Layer (Acceso a Datos)
```
@Repository
public class UsuarioRepository {
    private static final Logger logger = 
        LoggerFactory.getLogger(UsuarioRepository.class);
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public Usuario save(Usuario usuario) {
        try {
            logger.debug("Guardando usuario: {}", usuario.getEmail());
            
            if (usuario.getId() == null) {
                entityManager.persist(usuario);
                logger.debug("Usuario persistido: {}", usuario.getId());
            } else {
                usuario = entityManager.merge(usuario);
                logger.debug("Usuario actualizado: {}", usuario.getId());
            }
            
            return usuario;
            
        } catch (PersistenceException e) {
            logger.error("Error de persistencia guardando usuario {}", 
                        usuario.getEmail(), e);
            
            if (e.getCause() instanceof ConstraintViolationException) {
                logger.warn("Violación de constraint para usuario {}", 
                           usuario.getEmail());
                throw new DuplicadoException("Usuario duplicado", e);
            }
            throw new DataAccessException("Error de base de datos", e);
        }
    }
    
    public Optional<Usuario> findByEmail(String email) {
        logger.trace("Buscando usuario por email: {}", email);
        
        try {
            TypedQuery<Usuario> query = entityManager.createQuery(
                "SELECT u FROM Usuario u WHERE u.email = :email", Usuario.class);
            query.setParameter("email", email);
            
            Usuario usuario = query.getSingleResult();
            logger.debug("Usuario encontrado: {} - {}", usuario.getId(), email);
            
            return Optional.of(usuario);
            
        } catch (NoResultException e) {
            logger.debug("Usuario no encontrado para email: {}", email);
            return Optional.empty();
            
        } catch (Exception e) {
            logger.error("Error buscando usuario por email: {}", email, e);
            throw new DataAccessException("Error en consulta", e);
        }
    }
}
```
7. Utilidades/Helpers
```
public class SecurityUtil {
    private static final Logger logger = 
        LoggerFactory.getLogger(SecurityUtil.class);
    
    public static boolean validarToken(String token) {
        logger.trace("Validando token: {}...", 
                    token != null ? token.substring(0, Math.min(10, token.length())) : "null");
        
        if (token == null || token.trim().isEmpty()) {
            logger.warn("Token nulo o vacío recibido");
            return false;
        }
        
        try {
            // Validación compleja
            boolean valido = realizarValidacion(token);
            
            if (!valido) {
                logger.warn("Token inválido recibido");
            } else {
                logger.debug("Token validado exitosamente");
            }
            
            return valido;
            
        } catch (Exception e) {
            logger.error("Error validando token", e);
            return false;
        }
    }
    
    public static void registrarIntentoFallido(String username, String ip) {
        logger.warn("Intento fallido de autenticación - Usuario: {}, IP: {}", 
                   username, ip);
        
        // Lógica de bloqueo después de N intentos
        int intentos = incrementarIntentos(username);
        
        if (intentos >= MAX_INTENTOS) {
            logger.warn("Usuario {} bloqueado por {} intentos fallidos", 
                       username, intentos);
            bloquearUsuario(username);
        }
    }
}
```

9. Configuración por Ambiente (application.yml)
```# application.yml
logging:
  level:
    root: INFO
    com.empresa.proyecto: DEBUG
    org.springframework.web: INFO
    org.hibernate.SQL: DEBUG  # Ver SQLs en desarrollo
    org.hibernate.type: TRACE # Ver parámetros SQL en desarrollo
    
  # Perfiles por ambiente
---
spring:
  profiles: dev
  
logging:
  level:
    com.empresa.proyecto: TRACE
    org.hibernate.SQL: DEBUG
    org.hibernate.type: TRACE
    
  file:
    name: ./logs/app-dev.log
    
  logback:
    config: classpath:logback-dev.xml
    
---
spring:
  profiles: prod
  
logging:
  level:
    com.empresa.proyecto: INFO
    org.hibernate.SQL: WARN
    org.hibernate.type: WARN
    
  file:
    name: /var/log/app/app-prod.log
    
  logback:
    config: classpath:logback-prod.xml
```

12. Manejo de Excepciones con Logging
```
@ControllerAdvice
public class GlobalExceptionHandler {
    private static final Logger logger = 
        LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(BusinessException.class)
    public String handleBusinessException(BusinessException e, 
                                          HttpServletRequest request) {
        
        logger.warn("Excepción de negocio: {} - URL: {}", 
                   e.getMessage(), request.getRequestURL());
        
        // Log adicional para debugging
        logger.debug("Stack trace de negocio: ", e);
        
        return "error-business";
    }
    
    @ExceptionHandler(DataAccessException.class)
    public String handleDataAccessException(DataAccessException e) {
        logger.error("Error de acceso a datos", e);
        
        // Datos sensibles - no loggear todo
        String mensajeSeguro = e.getMessage() != null ? 
            e.getMessage().replaceAll("password=.*?(?=&|$)", "password=***") : 
            "Error de base de datos";
            
        logger.error("Error procesado: {}", mensajeSeguro);
        
        return "error-database";
    }
    
    @ExceptionHandler(Exception.class)
    public String handleGenericException(Exception e, 
                                         HttpServletRequest request) {
        
        // Log completo con contexto
        logger.error("""
            Error no manejado:
            URL: {}
            Método: {}
            IP: {}
            User Agent: {}
            Error: {}""",
            request.getRequestURL(),
            request.getMethod(),
            request.getRemoteAddr(),
            request.getHeader("User-Agent"),
            e.getMessage(),
            e);
        
        // Notificar a operaciones (Sentry, etc.)
        notificarErrorCritico(e, request);
        
        return "error-general";
    }
}

```
18. Ejemplo de Clase Perfectamente Loggeada:
```
@Service
public class FacturaService {
    private static final Logger logger = 
        LoggerFactory.getLogger(FacturaService.class);
    
    public Factura emitirFactura(FacturaDTO facturaDTO) {
        logger.info("Iniciando emisión de factura para cliente {}", 
                   facturaDTO.getClienteId());
        logger.debug("Datos factura: {}", facturaDTO);
        
        try {
            // Validación
            validarFactura(facturaDTO);
            
            // Persistencia
            logger.debug("Persistiendo factura en BD");
            Factura factura = facturaRepository.save(convertir(facturaDTO));
            
            // Operaciones secundarias
            logger.debug("Generando PDF de factura {}", factura.getId());
            generarPDF(factura);
            
            logger.info("Factura {} emitida exitosamente para cliente {}", 
                       factura.getId(), facturaDTO.getClienteId());
            
            return factura;
            
        } catch (ValidacionException e) {
            logger.warn("Validación fallida para factura cliente {}: {}", 
                       facturaDTO.getClienteId(), e.getMessage());
            throw e;
            
        } catch (Exception e) {
            logger.error("Error crítico emitiendo factura para cliente {}", 
                        facturaDTO.getClienteId(), e);
            throw new ServiceException("Error emitiendo factura", e);
        }
    }
    
    private void validarFactura(FacturaDTO facturaDTO) {
        logger.trace("Validando factura");
        
        if (facturaDTO.getMontoTotal().compareTo(BigDecimal.ZERO) <= 0) {
            logger.warn("Monto inválido en factura: {}", 
                       facturaDTO.getMontoTotal());
            throw new ValidacionException("Monto inválido");
        }
        
        logger.trace("Factura validada exitosamente");
    }
}
```




# Mapeo DTO ↔ Entity (Correctamente)
2. Mapeo Manual (Alternativa)
 ```
@Component
public class UsuarioMapper {
    
    public UsuarioDTO toDto(Usuario entity) {
        if (entity == null) {
            return null;
        }
        
        UsuarioDTO dto = new UsuarioDTO();
        dto.setId(entity.getId());
        dto.setNombre(entity.getNombre());
        dto.setEmail(entity.getEmail());
        dto.setTelefono(entity.getTelefono());
        dto.setRfc(entity.getRfc());
        dto.setEdad(entity.getEdad());
        dto.setSalario(entity.getSalario());
        dto.setActivo(entity.isActivo());
        dto.setFechaNacimiento(entity.getFechaNacimiento());
        dto.setFechaCreacion(entity.getFechaCreacion());
        dto.setFechaModificacion(entity.getFechaModificacion());
        
        // Mapeo de relaciones
        if (entity.getDireccion() != null) {
            dto.setDireccion(direccionMapper.toDto(entity.getDireccion()));
        }
        
        if (entity.getRoles() != null) {
            dto.setRoles(rolMapper.toDtoList(entity.getRoles()));
        }
        
        return dto;
    }
    
    public Usuario toEntity(UsuarioDTO dto) {
        if (dto == null) {
            return null;
        }
        
        Usuario entity = new Usuario();
        updateEntity(dto, entity);
        return entity;
    }
    
    public void updateEntity(UsuarioDTO dto, Usuario entity) {
        // Solo actualizar campos permitidos
        entity.setNombre(dto.getNombre());
        entity.setEmail(dto.getEmail().toLowerCase());
        entity.setTelefono(dto.getTelefono().replaceAll("[^0-9]", ""));
        entity.setRfc(dto.getRfc().toUpperCase());
        entity.setEdad(dto.getEdad());
        entity.setSalario(dto.getSalario());
        entity.setActivo(dto.isActivo());
        entity.setFechaNacimiento(dto.getFechaNacimiento());
        entity.setFechaModificacion(LocalDateTime.now());
        
        // No actualizar: id, fechaCreacion, version (optimistic locking)
    }
    
    public List<UsuarioDTO> toDtoList(List<Usuario> entities) {
        return entities.stream()
            .map(this::toDto)
            .collect(Collectors.toList());
    }
}
 ```
13. Service con Mapeo

 ```
@Service
@Transactional
@Slf4j
public class UsuarioService {
    
    @Autowired
    private UsuarioRepository usuarioRepository;
    
    @Autowired
    private UsuarioMapper usuarioMapper;
    
    @Autowired
    private Validator validator;
    
    public UsuarioDTO crearUsuario(@Valid UsuarioDTO usuarioDTO) {
        // Validar
        validarUsuarioDTO(usuarioDTO);
        
        // Convertir DTO a Entity
        Usuario usuario = usuarioMapper.toEntity(usuarioDTO);
        
        // Setear campos de auditoría
        usuario.setFechaCreacion(LocalDateTime.now());
        usuario.setCreadoPor(obtenerUsuarioActual());
        
        // Guardar
        usuario = usuarioRepository.save(usuario);
        
        // Retornar DTO
        return usuarioMapper.toDto(usuario);
    }
    
    public UsuarioDTO actualizarUsuario(Long id, @Valid UsuarioDTO usuarioDTO) {
        // Buscar entidad existente
        Usuario usuario = usuarioRepository.findById(id)
            .orElseThrow(() -> new EntityNotFoundException("Usuario no encontrado"));
        
        // Validar
        validarUsuarioDTO(usuarioDTO);
        
        // Actualizar solo campos permitidos
        usuarioMapper.updateEntity(usuarioDTO, usuario);
        usuario.setFechaModificacion(LocalDateTime.now());
        usuario.setModificadoPor(obtenerUsuarioActual());
        
        // Guardar
        usuario = usuarioRepository.save(usuario);
        
        return usuarioMapper.toDto(usuario);
    }
    
    public Page<UsuarioDTO> buscarUsuarios(UsuarioFiltro filtro, Pageable pageable) {
        Page<Usuario> pagina = usuarioRepository.findByFiltro(filtro, pageable);
        
        // Convertir página completa
        return pagina.map(usuarioMapper::toDto);
    }
    
    private void validarUsuarioDTO(UsuarioDTO usuarioDTO) {
        Set<ConstraintViolation<UsuarioDTO>> violations = 
            validator.validate(usuarioDTO);
        
        if (!violations.isEmpty()) {
            throw new ConstraintViolationException(violations);
        }
        
        // Validaciones de negocio adicionales
        if (usuarioRepository.existsByEmail(usuarioDTO.getEmail())) {
            throw new BusinessValidationException("Email ya registrado");
        }
    }
}
 ```
14. Patrón Builder para Entidades Complejas
 ```
// Entity con Builder
@Entity
@Table(name = "usuarios")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Usuario {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank
    @Size(max = 100)
    private String nombre;
    
    @NotBlank
    @Email
    @Column(unique = true)
    private String email;
    
    // Método de creación
    public static Usuario crearDesdeDTO(UsuarioDTO dto) {
        return Usuario.builder()
            .nombre(dto.getNombre())
            .email(dto.getEmail().toLowerCase())
            .telefono(dto.getTelefono().replaceAll("[^0-9]", ""))
            .rfc(dto.getRfc().toUpperCase())
            .edad(dto.getEdad())
            .salario(dto.getSalario())
            .activo(true)
            .fechaCreacion(LocalDateTime.now())
            .build();
    }
    
    public void actualizarDesdeDTO(UsuarioDTO dto) {
        this.nombre = dto.getNombre();
        this.email = dto.getEmail().toLowerCase();
        this.telefono = dto.getTelefono().replaceAll("[^0-9]", "");
        this.rfc = dto.getRfc().toUpperCase();
        this.edad = dto.getEdad();
        this.salario = dto.getSalario();
        this.fechaModificacion = LocalDateTime.now();
    }
}
 ```
15. Mapeo para Relaciones


 ```
// Mapeo OneToMany
@Entity
public class Usuario {
    
    @OneToMany(mappedBy = "usuario", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Direccion> direcciones = new ArrayList<>();
    
    public void agregarDireccion(DireccionDTO direccionDTO) {
        Direccion direccion = DireccionMapper.INSTANCE.toEntity(direccionDTO);
        direccion.setUsuario(this);
        this.direcciones.add(direccion);
    }
    
    public void actualizarDirecciones(List<DireccionDTO> dtos) {
        // Limpiar y agregar nuevas
        this.direcciones.clear();
        dtos.forEach(this::agregarDireccion);
    }
}

// DTO con relaciones
public class UsuarioDTO {
    private List<DireccionDTO> direcciones;
    
    // Para formularios anidados
    private DireccionDTO nuevaDireccion = new DireccionDTO();
}
 ```



