=================
Content types
=================

.. admonition:: Description

	Plone's content type subsystems and creating new content types programmatically.

.. contents :: :local:

Introduction
-------------

Plone has two kind of content types subsystems

* :doc:`Archetypes </content/archetypes/index>` based

* :doc:`Dexterity </content/dexterity>` based (new)

* See also Plomino (later in this document)

Flexible architecture allows other kinds of content type subsystems as well.

Type information registry
-------------------------

Plone maintains available content types in portal_types tool.

portal_types is a folderish content where type information are child objects,
keyed by portal_type metadata.

portal_factory is a tool responsible for creating the persistent object representing the content.

`TypesTool source code <http://svn.zope.org/Products.CMFCore/trunk/Products/CMFCore/TypesTool.py?rev=101748&view=auto>`_.

Listing available content types
================================

Many times you need to ask the user to choose specific Plone content types.

Plone offers two Zope 3 vocabularies for this purpose

* *plone.app.vocabularies.PortalTypes*, a list of types installed in portal_types

* *plone.app.vocabularies.ReallyUserFriendlyTypes*, a list of those types that are likely to mean something to users
      
Below is how to do ask vocabularie with raw Python code::

        
        from Acquisition import aq_inner
        from zope.app.component.hooks import getSite
        from zope.schema.vocabulary import SimpleVocabulary, SimpleTerm
        from Products.CMFCore.utils import getToolByName

        def friendly_types(site):
            """ List user selectable content types.
            
            Note that there exist a method in IPortalState utility view for this, but we cannot
            use it, because vocabulary factory must be available in contexts where there is
            no HTTP request (e.g. installing add-on product).
             
            This code is copy-pasted from https://svn.plone.org/svn/plone/plone.app.layout/trunk/plone/app/layout/globals/portal.py
            
            @return: Generator for (id, type_info title) tuples
            """
            context = aq_inner(site)
            site_properties = getToolByName(context, "portal_properties").site_properties
            not_searched = site_properties.getProperty('types_not_searched', [])
        
            portal_types = getToolByName(context, "portal_types")
            types = portal_types.listContentTypes()
            
            # Get list of content type ids which are not filtered out
            prepared_types = [t for t in types if t not in not_searched]
            
            # Return (id, title) pairs
            return [ (id, portal_types[id].title) for id in prepared_types ]
        
Creating a new content type
----------------------------

This instructions apply for :doc:`Archetypes subsystem based content types </content/archetypes/index>`

* You need to have an add-on product code skeleton created using paster's *archetypes* template

* Use *paster addcontent content* command new types. 

Related how tos

* http://lionfacelemonface.wordpress.com/tutorials/zopeskel-archetypes-howto/

* http://docs.openia.com/howtos/development/plone/creating-a-site-archetypes-object-and-contenttypes-with-paster?set_language=fi&cl=fi

* http://www.unc.edu/~jj/plone/

.. note ::

        Creating types by hand is not worth of the problems. Please use a 
        code generator to create the skeleton for your new content type.

.. warning::

        Content type name must not contain spaces. Content type name or description
        must not contain non-ASCII letters. If you need to change these please
        create a translation catalog which will translate the text to 
        one with spaces or international letters.  

Debugging new content type problems
===================================

Creating types by hand is not worth of the problems.

* `Why doesn't my custom content type show up in add menu <http://plone.org/documentation/faq/why-doesnt-my-custom-content-type-show-up-in-add-menu/>`_ checklist.

Creating new content types through-the-web
---------------------------------------------

There exist solutions for non-programmes and Plone novices to create their content types
more easily.

Dexterity 

* http://plone.org/products/dexterity

* Core feature

* Use Dexterity control panel in site setup

Plomino (Archetypes-based add-on)

* With Plomino you can make an entire web application that can organize &
  manipulate data with very limited programming experience.

* http://www.plomino.net/

* http://www.youtube.com/view_play_list?p=469DE37C742F31D1

Implictly allowed
------------------

Implictly allowed is flag whether the content is globally addable or
must be specifically enabled for certain folders.

The following example allows creation of Large Plone Folder anywhere at the site
(it is disabled by default). For available properties, see TypesTool._advanced_properties.

Example::

    portal_types = self.context.portal_types
    lpf = portal_types["Large Plone Folder"]
    lpf.global_allow = True # This is "Globally allowed" property


Constraining the addable types per type instance
------------------------------------------------

For the instances of some content types, the user may manually
restrict which kinds of objects may be added inside. This is done by clicking
the *Add new...* link on the green edit bar and then choosing
*Restrictions...*.
 
This can also be done programmatically on an instance of a content type that
supports it.

First, we need to know whether the instance supports this:

Example:: 

    from Products.Archetypes.utils import shasattr # To avoid acquisition
    if shasattr(context, 'canSetConstrainTypes'):
        # constrain the types
        context.setConstrainTypesMode(1)
        context.setLocallyAllowedTypes(('News Item',))

If setConstrainTypesMode is 1, then only the types enabled by using
setLocallyAllowedTypes will be allowed.

The types specified by setLocallyAllowedTypes must be a subset of the allowable
types specified in the content-type's FTI (Factory Type Information) in the
portal_types tool.

If you want the types to appear in the "Add new.." dropdown menu, then you must
also set the immediately addable types. Otherwise, they will appear under the
"more" submenu of "Add new..".

Example::

    context.setImmediatelyAddableTypes(('News Item',))

The immediately addable types must be a subset of the locally allowed types.


To retrieve information on the constrained types, you can just use the accessor
equivalents of the above methods.

Example::

    context.getConstrainTypesMode()
    context.getLocallyAllowedTypes()
    context.getImmediatelyAddableTypes()
    context.getDefaultAddableTypes()
    context.allowedContentTypes()

**Be careful of Acquisition**. You might be aquiring these methods from the
current instance's parent. It would be wise to first check whether the current
object has this attibute. Either by using *shasattr* or by using *hasattr* on the
object's base (via *aq_base*).

The default addable types, are the types that would be addable, if
constrainTypesMode is 0 (i.e not enabled).

For more information, see **Products/CMFPlone/interfaces/constraints.py**

