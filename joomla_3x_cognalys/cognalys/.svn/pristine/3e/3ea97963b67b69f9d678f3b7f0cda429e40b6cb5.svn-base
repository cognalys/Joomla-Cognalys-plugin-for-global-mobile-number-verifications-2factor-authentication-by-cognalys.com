<?php
// Check to ensure this file is included in Joomla!
defined('_JEXEC') or die();
jimport('joomla.form.formfield');
class JFormFieldCognalyshelp extends JFormField 
{
	/**
	 * Element name
	 *
	 * @access	protected
	 * @var		string
	 */
	
	protected $type = 'previewcaptchawithsettings';
	
	protected function getInput()
	{
		$elementData = '<p>Please, create a cognalys account ( <a target="_new" href="https://www.cognalys.com/">https://www.cognalys.com/</a> ).<br/>
To create new OTP Application, click DASHBORD -> OTP Applications -> Create new.</p>
<img src="' . JURI::base() . '../plugins/system/cognalys/include/img/new_otp_app.png"/>
<p>To grab APP ID and Access Token of your OTP App, click DASHBORD -> OTP Applications -> Manage.</p>
<img src="' . JURI::base() . '../plugins/system/cognalys/include/img/manage_otp_app.png"/>
<p>And click Configuration from panel of you OTP App.</p>
<img src="' . JURI::base() . '../plugins/system/cognalys/include/img/config_otp_app.png"/>
';
		
		return $elementData;
	}
}