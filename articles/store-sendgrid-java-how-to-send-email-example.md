<properties linkid="store-requestform-preview" urlDisplayName="Request Azure Store Integration" pageTitle="How to Send Email Using SendGrid from Java in a Windows Azure Deployment" metaKeywords="" description="" metaCanonical="" services="" documentationCenter="" title="How to Send Email Using SendGrid from Java in a Windows Azure Deployment" authors=""  solutions="" writer="waltpo" manager="" editor="mollybos"  />





# How to Send Email Using SendGrid from Java in a Windows Azure Deployment

The following example shows you how you can use SendGrid to send emails from a web page hosted in Windows Azure. The resulting application will prompt the user for email values, as shown in the following screen shot.

![Email form][emailform]

The resulting email will look similar to the following screen shot.

![Email message][emailsent]

You'll need to do the following to use the code in this topic:

1. Obtain the javax.mail JARs, for example from <http://www.oracle.com/technetwork/java/javamail/index.html>.
2. Add the JARs to your Java build path.
3. If you are using Eclipse to create this Java application, you can include the SendGrid libraries in your application deployment file (WAR) using Eclipse's deployment assembly feature. If you are not using Eclipse to create this Java application, ensure the libraries are included within the same Azure role as your Java application, and added to the class path of your application.


You must also have your own SendGrid username and password, to be able to send the email. To get started with SendGrid, see [How to send email using SendGrid from Java](/en-us/develop/java/how-to-guides/sendgrid-email-service/).

Additionally, familiarity with the information at [Creating a Hello World Application for Windows Azure in Eclipse](http://msdn.microsoft.com/en-us/library/windowsazure/hh690944), or with other techniques for hosting Java applications in Windows Azure if you are not using Eclipse, is highly recommended.

## Create a web form for sending email

The following code shows how to create a web form to retrieve user data for sending email. For purposes of this content, the JSP file is named **emailform.jsp**.

	<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
	    pageEncoding="ISO-8859-1" %>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Email form</title>
	</head>
	<body>
	 <p>Fill in all fields and click <b>Send this email</b>.</p>
	 <br/>
	  <form action="sendemail.jsp" method="post">
	   <table>
	     <tr>
	       <td>To:</td>
	       <td><input type="text" size=50 name="emailTo">
	       </td>
	     </tr>
	     <tr>
	       <td>From:</td>
	       <td><input type="text" size=50 name="emailFrom">
	       </td>
	     </tr>
	     <tr>
	       <td>Subject:</td>
	       <td><input type="text" size=100 name="emailSubject" value="My email subject">
	       </td>
	     </tr>
	     <tr>
	       <td>Text:</td>
	       <td><input type="text" size=400 name="emailText" value="Hello,<p>This is my message.</p>Thank you." />
	       </td>
	     </tr>
	     <tr>
	       <td>SendGrid user name:</td>
	       <td><input type="text" name="sendGridUser">
	       </td>
	     </tr>
	     <tr>
	       <td>SendGrid password:</td>
	       <td><input type="password" name="sendGridPassword">
	       </td>
	     </tr>
	     <tr>
	       <td colspan=2><input type="submit" value="Send this email">
	       </td>
	     </tr>
	   </table>
	 </form>
	 <br/>
	</body>
	</html>

## Create the code to send the email

The following code, which is called when you complete the form in emailform.jsp, creates the email message and sends it. For purposes of this content, the JSP file is named **sendemail.jsp**.

	<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
	    pageEncoding="ISO-8859-1" import="javax.activation.*, javax.mail.*, javax.mail.internet.*, java.util.Date, java.util.Properties" %>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Email processing happens here</title>
	</head>
	<body>
	    <b>This is my send mail page.</b><p/>
	 <%
	 
	 final String sendGridUser = request.getParameter("sendGridUser");
	 final String sendGridPassword = request.getParameter("sendGridPassword");
	 
	 class SMTPAuthenticator extends Authenticator
	 {
	   public PasswordAuthentication getPasswordAuthentication()
	   {
	        String username = sendGridUser;
	        String password = sendGridPassword;
	      
	        return new PasswordAuthentication(username, password);   
	   }
	 }
	 try
	 {
	     
	     // The SendGrid SMTP server.
	     String SMTP_HOST_NAME = "smtp.sendgrid.net";
	
	     Properties properties;
	    
	     properties = new Properties();
	     
	     // Specify SMTP values.
	     properties.put("mail.transport.protocol", "smtp");
	     properties.put("mail.smtp.host", SMTP_HOST_NAME);
	     properties.put("mail.smtp.port", 587);
	     properties.put("mail.smtp.auth", "true");
	     
	     // Display the email fields entered by the user. 
	     out.println("Value entered for email Subject: " + request.getParameter("emailSubject") + "<br/>");        
	     out.println("Value entered for email      To: " + request.getParameter("emailTo") + "<br/>");
	     out.println("Value entered for email    From: " + request.getParameter("emailFrom") + "<br/>");
	     out.println("Value entered for email    Text: " + "<br/>" + request.getParameter("emailText") + "<br/>");
	
	     // Create the authenticator object.
	     Authenticator authenticator = new SMTPAuthenticator();
	     
	     // Create the mail session object.
	     Session mailSession;
	     mailSession = Session.getDefaultInstance(properties, authenticator);
	     
	     // Display debug information to stdout, useful when using the
	     // compute emulator during development.
	     mailSession.setDebug(true);
	
	     // Create the message and message part objects.
	     MimeMessage message;
	     Multipart multipart;
	     MimeBodyPart messagePart; 
	     
	     message = new MimeMessage(mailSession);
	     
	     multipart = new MimeMultipart("alternative");
	     messagePart = new MimeBodyPart();
	     messagePart.setContent(request.getParameter("emailText"), "text/html");
	     multipart.addBodyPart(messagePart);            
	
	     // Specify the email To, From, Subject and Content. 
	     message.setFrom(new InternetAddress(request.getParameter("emailFrom")));
	     message.addRecipient(Message.RecipientType.TO, new InternetAddress(request.getParameter("emailTo")));
	     message.setSubject(request.getParameter("emailSubject")); 
	     message.setContent(multipart);
	     
	     // Uncomment the following if you want to add a footer.
	     // message.addHeader("X-SMTPAPI", "{\"filters\": {\"footer\": {\"settings\": {\"enable\":1,\"text/html\": \"<html>This is my <b>email footer</b>.</html>\"}}}}");
	
	     // Uncomment the following if you want to enable click tracking.
	     // message.addHeader("X-SMTPAPI", "{\"filters\": {\"clicktrack\": {\"settings\": {\"enable\":1}}}}");
	     
	     Transport transport;
	     transport = mailSession.getTransport();
	     // Connect the transport object.
	     transport.connect();
	     // Send the message.
	     transport.sendMessage(message,  message.getRecipients(Message.RecipientType.TO));
	     // Close the connection.
	     transport.close();
	 
	    out.println("<p>Email processing completed.</p>");
	     
	 }
	 catch (Exception e)
	 {
	     out.println("<p>Exception encountered: " + 
	                        e.getMessage()     +
	                        "</p>");   
	 }
	%>
	
	</body>
	</html>

In addition to sending the email, emailform.jsp provides a result for the user; an example is the following screen shot:

![Send mail result][emailresult]

## Next steps

Deploy your application to the compute emulator and within a browser run emailform.jsp, enter values in the form, click **Send this email**, and then see results in sendemail.jsp.

This code was provided to show you how to use SendGrid in Java on Windows Azure. Before deploying to Windows Azure in production, you may want to add more error handling or other features. For example: 

* You could use Windows Azure storage blobs or SQL Database to store email addresses and email messages, instead of using a web form. For information about using Windows Azure storage blobs in Java, see [How to Use the Blob Storage Service from Java](http://www.windowsazure.com/en-us/develop/java/how-to-guides/blob-storage/). For information about using SQL Database in Java, see [Using SQL Database in Java](http://www.windowsazure.com/en-us/develop/java/how-to-guides/using-sql-azure-in-java/).
* You could use `RoleEnvironment.getConfigurationSettings` to retrieve the SendGrid username and password from your deployment's configuration settings, instead of using the web form to retrieve those values. For information about the `RoleEnvironment` class, see [Using the Windows Azure Service Runtime Library in JSP](http://msdn.microsoft.com/en-us/library/windowsazure/hh690948) and the Windows Azure Service Runtime package documentation at <http://dl.windowsazure.com/javadoc>.
* For more information about using SendGrid in Java, see [How to Send Email Using SendGrid from Java](./sendgrid-email-service.md).

[emailform]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaEmailform.jpg
[emailsent]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaEmailSent.jpg
[emailresult]: ./media/store-sendgrid-java-how-to-send-email-example/SendGridJavaResult.jpg
