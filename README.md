#Java Persistence API - general project for the OrgApp

This project consists of the dependencies for JPA and Hibernate. 

The Maven dependency for this project is: 

```Maven
    <dependency>
	  <groupId>com.orgapp.jpa</groupId>
	  <artifactId>orgappJPA</artifactId>
	  <version>1.0-SNAPSHOT</version>      
    </dependency>  
```

##Usage of the JPA

In every sub-project, a persistence.xml document must be created. The document has to be stored in the resources/META-INF directory. 

The document specifies the so called persistence-units which can be referenced from code using an EntityManagerFactory. 

An example persistence.xml document may look as follows:

```xml
<!--
  ~ Hibernate, Relational Persistence for Idiomatic Java
  ~
  ~ License: GNU Lesser General Public License (LGPL), version 2.1 or later.
  ~ See the lgpl.txt file in the root directory or <http://www.gnu.org/licenses/lgpl-2.1.html>.
  -->
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0">

    <persistence-unit name="org.hibernate.orgapp.jpa">
        <description>
            General JPA (Java Persistence API) project for the OrgApp
        </description>

		<class>org.jpa.client.orgapp.Event</class>

        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:db1;DB_CLOSE_DELAY=-1;MVCC=TRUE" />
            <property name="javax.persistence.jdbc.user" value="sa" />
            <property name="javax.persistence.jdbc.password" value="" />

            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>

    </persistence-unit>

</persistence>

```

###Persistence-Unit

The persistence unit must carry a unique name:

```xml
<persistence-unit name="org.hibernate.orgapp.jpa">
```

###class
Each persistence entity must be listed here with a reference to the actual class in the project.

```
<class>org.jpa.client.orgapp.Event</class>
```


##JPA Usage Example 
This is a simple example how to create one instance of EntityManagerFactory and create an EntityManager

```java

	private EntityManagerFactory entityManagerFactory;

	@Override
	protected void setUp() throws Exception {
		// like discussed with regards to SessionFactory, an EntityManagerFactory is set up once for an application
		// 		IMPORTANT: notice how the name here matches the name we gave the persistence-unit in persistence.xml!
		entityManagerFactory = Persistence.createEntityManagerFactory( "org.hibernate.orgapp.jpa" );
	}

	@Override
	protected void tearDown() throws Exception {
		entityManagerFactory.close();
	}

	public void testBasicUsage() {
		// create a couple of events...
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		entityManager.getTransaction().begin();
		entityManager.persist( new Event( "Our very first event!", new Date() ) );
		entityManager.persist( new Event( "A follow up event", new Date() ) );
		entityManager.getTransaction().commit();
		entityManager.close();

		// now lets pull events from the database and list them
		entityManager = entityManagerFactory.createEntityManager();
		entityManager.getTransaction().begin();
        List<Event> result = entityManager.createQuery( "from Event", Event.class ).getResultList();
		for ( Event event : result ) {
			System.out.println( "Event (" + event.getDate() + ") : " + event.getTitle() );
		}
        entityManager.getTransaction().commit();
        entityManager.close();
	}
```
		
The basic Event class looks as follows:

```Java
package org.jpa.client.orgapp;

import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

@Entity
@Table( name = "EVENTS" )
public class Event {
    private Long id;

    private String title;
    private Date date;

	public Event() {
		// this form used by Hibernate
	}

	public Event(String title, Date date) {
		// for application use, to create new events
		this.title = title;
		this.date = date;
	}

	@Id
	@SequenceGenerator(name="SEQ_GEN", initialValue = 1, sequenceName="SEQ_JUST_FOR_TEST", allocationSize=1)
	@GeneratedValue(strategy=GenerationType.SEQUENCE, generator="SEQ_GEN")
    public Long getId() {
		return id;
    }

    private void setId(Long id) {
		this.id = id;
    }

	@Temporal(TemporalType.TIMESTAMP)
	@Column(name = "EVENT_DATE")
    public Date getDate() {
		return date;
    }

    public void setDate(Date date) {
		this.date = date;
    }

    public String getTitle() {
		return title;
    }

    public void setTitle(String title) {
		this.title = title;
    }
}
```


##Further information
Interesting links with more detailed information about JPA and how to configure the persistence.xml:

https://docs.jboss.org/hibernate/entitymanager/3.5/reference/en/html/configuration.html

http://www.torsten-horn.de/techdocs/java-jpa.htm#JpaJavaSE-persistence.xml
